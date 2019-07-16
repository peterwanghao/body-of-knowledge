# 1 用Redis作为Mysql数据库的缓存

​		用Redis作**MySQL数据库**缓存，必须解决2个问题。首先，应该确定用何种**数据结构**存储来自Mysql的数据；在确定数据结构之后，还要考虑用什么标识作为该数据结构的键。

​        直观上看，Mysql中的数据都是按表存储的；更微观地看，这些表都是按行存储的。每执行一次select查询，Mysql都会返回一个结果集，这个结果集由若干行组成。所以，一个自然而然的想法就是在Redis中找到一种对应于Mysql行的数据结构。Redis中提供了五种基本数据结构，即字符串（string）、列表（list）、哈希（hash）、集合（set）和有序集合（sorted set）。经过调研，发现适合存储行的数据结构有两种，即string和hash。

​        要把Mysql的行数据存入string，首先需要对行数据进行格式化。事实上，结果集的每一行都可以看做若干由字段名和其对应值组成的键值对集合。这种键值对结构很容易让我们想起Json格式。因此，这里选用Json格式作为结果集每一行的格式化模板。根据这一想法，我们可以实现将结果集格式化为若干Json对象，并将Json对象转化为字符串存入Redis的代码：

```
// 该函数把结果集中的每一行转换为一个Json格式的字符串并存入Redis的STRING结构中，
// STRING键应该包含结果集标识符和STRING编号，形式如“cache.string:123456:1”
string Cache2String(sql::Connection *mysql_connection,
                    redisContext *redis_connection,
                    sql::ResultSet *resultset,
                    const string &resultset_id, int ttl) {
  if (resultset->rowsCount() == 0) {
    throw runtime_error("FAILURE - no rows");
  }
  // STRING键的前缀，包含了结果集的标识符
  string prefix("cache.string:" + resultset_id + ":");
  unsigned int num_row = 1;  // STRING编号，附加于STRING键的末尾，从1开始
  sql::ResultSetMetaData *meta = resultset->getMetaData();
  unsigned int num_col = meta->getColumnCount();
  // 将结果集中所有行对应的所有STRING键存入该SET，SET键包含了结果集的标识符
  string redis_row_set_key("resultset.string:" + resultset_id);
  redisReply *reply;
  string ttlstr;
  stringstream ttlstream;
  ttlstream << ttl;
  ttlstr = ttlstream.str();
  resultset->beforeFirst();
  // 将结果集中的每一行转为Json格式的字符串，将这些Json字符串存入STRING，
  // 每个STRING对应结果集中的一行
  while (resultset->next()) {
    string redis_row_key;  // STRING键名，由前缀和STRING编号组成
    stringstream keystream;
    keystream << prefix << num_row;
    redis_row_key = keystream.str();
    Json::Value row;
    for (int i = 1; i <= num_col; ++i) {
      string col_label = meta->getColumnLabel(i);
      string col_value = resultset->getString(col_label);
      row[col_label] = col_value;
    }
    Json::FastWriter writer;
    string redis_row_value = writer.write(row);
	// 将STRING键及Json格式的对应值对存入Redis
    reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                                 "SET %s %s",
                                                 redis_row_key.c_str(), 
                                                 redis_row_value.c_str()));
    freeReplyObject(reply);
    // 将STRING键加入SET中
    reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                                 "SADD %s %s",
                                                 redis_row_set_key.c_str(), 
                                                 redis_row_key.c_str()));
    freeReplyObject(reply);
    // 设置STRING的过期时间
    reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                                 "EXPIRE %s %s",
                                                 redis_row_key.c_str(), 
                                                 ttlstr.c_str()));
    freeReplyObject(reply);
    ++num_row;
  }
  // 设置SET的过期时间
  reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                               "EXPIRE %s %s",
                                               redis_row_set_key.c_str(), 
                                               ttlstr.c_str()));
  freeReplyObject(reply);
  return redis_row_set_key;  // 返回SET键，以便于其他函数获取该SET中的内容
}
```

​		要把Mysql的行数据存入hash，过程要比把数据存入string直观很多。这是由hash的结构性质决定的——hash本身就是一个键值对集合：一个“父键”下面包含了很多“子键”，每个“子键”都对应一个值。根据前面的分析可知，结果集中的每一行实际上也是键值对集合。用Redis键值对集合表示Mysql键值对集合应该再合适不过了：对于结果集中的某一行，字段对应于hash的“子键”，字段对应的值就是hash“子键”对应的值，即结果集的一行刚好对应一个hash。这一想法的实现代码如下：

```
// 该函数把结果集中的每一行都存入一个HASH结构。HASH键应当包括结果集标识符和HASH编号，
// 形如“cache.string:123456:1”
string Cache2Hash(sql::Connection *mysql_connection,
                  redisContext *redis_connection,
                  sql::ResultSet *resultset,
                  const string &resultset_id, int ttl) {
  if (resultset->rowsCount() == 0) {
    throw runtime_error("FAILURE - no rows");
  }
  // HASH键的前缀，包含了结果集的标识符
  string prefix("cache.hash:" + resultset_id + ":");
  unsigned int num_row = 1;  // HASH编号，附加于HASH键的末尾，从1开始
  sql::ResultSetMetaData *meta = resultset->getMetaData();
  unsigned int num_col = meta->getColumnCount();
  // 将结果集中所有行对应的所有HASH键存入该SET，SET键包含了结果集的标识符
  string redis_row_set_key("resultset.hash:" + resultset_id);
  redisReply *reply;
  string ttlstr;
  stringstream ttlstream;
  ttlstream << ttl;
  ttlstr = ttlstream.str();
  // 结果集中的每一行对应于一个HASH，将结果集的所有行都存入相应HASH中
  resultset->beforeFirst();
  while (resultset->next()) {
    string redis_row_key;  // HASH键名，由前缀和HASH编号组成
    stringstream keystream;
    keystream << prefix << num_row;
    redis_row_key = keystream.str();
    for (int i = 1; i <= num_col; ++i) {
      string col_label = meta->getColumnLabel(i);
      string col_value = resultset->getString(col_label);
	  // 将结果集中一行的字段名和对应值存入HASH
      reply = static_cast<redisReply*>(redisCommand(redis_connection,
                                                   "HSET %s %s %s",
                                                   redis_row_key.c_str(), 
                                                   col_label.c_str(),
                                                   col_value.c_str()));
      freeReplyObject(reply);
    }
	// 将HASH键加入SET中
    reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                                 "SADD %s %s",
                                                 redis_row_set_key.c_str(), 
                                                 redis_row_key.c_str())); 
    freeReplyObject(reply);
	// 设置HASH的过期时间
    reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                                 "EXPIRE %s %s",
                                                 redis_row_key.c_str(), 
                                                 ttlstr.c_str()));
    freeReplyObject(reply);
    ++num_row;
  }
  // 设置SET的过期时间
  reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                               "EXPIRE %s %s",
                                               redis_row_set_key.c_str(), 
                                               ttlstr.c_str()));
  freeReplyObject(reply);
  return redis_row_set_key;  // 返回SET键，以便于其他函数获取该SET中的内容
}
```

​		把**MySQL**结果集缓存到Redis的字符串或哈希结构中以后，我们面临一个新的问题，即如何为这些字符串或哈希命名，也就是如何确定它们的键。因为这些**数据结构**所对应的行都属于某个结果集，假如可以找到一种唯一标识结果集的方法，那么只需为这些数据结构分配一个唯一的序号，然后把结果集标识符与该序号结合起来，就能唯一标识一个数据结构了。于是，为字符串和哈希命名的问题就转化为确定结果集标识符的问题。

​        经过调研，发现一种较为通用的确定结果集标识符的方法。正如我们所知道的，缓存在Redis中的结果集数据都是利用select等sql语句从Mysql中获取的。同样的查询语句会生成同样的结果集（这里暂时不讨论结果集中每条记录的顺序问题），这一性质刚好可以用来确定结果集的唯一标识符。当然，简单地把整个sql语句作为结果集标识符是不可取的，一个显而易见的理由是，未经处理的sql查询语句均包含若干空格，而Redis的键是不允许存在空格的。这时，我们需要一个可以把sql语句转换为唯一标识符的函数。通常，这一功能由散列函数完成，包括MD5，SHA系列等加密散列函数在内的很多**算法**均可达到这一目的。

​        确定结果集标识符之后，从Redis读数据或向Redis写数据的思路就很清晰了。对于一个sql语句格式的数据请求，首先计算该语句的MD5并据此得到结果集标识符，然后利用该标识符在Redis中查找该结果集。注意，结果集中的每一行都有一个相应的键，这些键都存储在一个Redis集合结构中。这个集合恰好对应了所需的结果集，所以，该集合的键必须包含结果集标识符。如果Redis中不存在这样一个集合，说明要找的结果集不在Redis中，所以需要执行相应的sql语句，在Mysql中查询到相应的结果集，然后按照上面所说的办法把结果集中的每一行以字符串或哈希的形式存入Redis。在Redis中查找相应结果集的代码如下：

```
// 该函数根据sql语句在Redis中查询相应的结果集，并返回结果集中每一行所对应的数据结构的键
vector<string> GetCache(sql::Connection *mysql_connection,
                      redisContext *redis_connection,
                      const string &sql, int ttl, int type) {
  vector<string> redis_row_key_vector;
  string resultset_id = md5(sql);  // 计算sql语句的md5，这是唯一标识结果集的关键
  // type==1时，该函数将查询相应的STRING集合或将结果集写入若干STRING
  string cache_type = (type == 1) ? "string" : "hash";
  // 根据type信息和结果集标识符合成SET键
  string redis_row_set_key = "resultset." + cache_type + ":" + resultset_id;
  redisReply *reply;
  // 尝试从reply中获取SET中保存的所有键
  reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                               "SMEMBERS %s",
                                               redis_row_set_key.c_str()));
  if (reply->type == REDIS_REPLY_ARRAY) {
	// 如果要找的SET不存在，说明Redis中没有相应的结果集，需要调用Cache2String或
	// Cache2Hash函数把数据从Mysql拉取到Redis中
    if (reply->elements == 0) {
      freeReplyObject(reply);
      sql::Statement *stmt = mysql_connection->createStatement();
      sql::ResultSet *resultset = stmt->executeQuery(sql);
      if (type == 1) {
        redis_row_set_key = Cache2String(mysql_connection, redis_connection,
                                         resultset, resultset_id, ttl);
      } else {
        redis_row_set_key = Cache2Hash(mysql_connection, redis_connection, 
                                       resultset, resultset_id, ttl);
      }
	  // 再次尝试从reply中获取SET中保存的所有键
      reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                                   "SMEMBERS %s",
                                                   redis_row_set_key.c_str()));
      delete resultset;
      delete stmt;
    }
	// 把SET中的每个STRING或HASH键存入redis_row_key_vector中
    string redis_row_key;
    for (int i = 0; i < reply->elements; ++i) {
      redis_row_key = reply->element[i]->str;
      redis_row_key_vector.push_back(redis_row_key);
    }
    freeReplyObject(reply);
  } else {
    freeReplyObject(reply);
    throw runtime_error("FAILURE - SMEMBERS error");
  }
  return redis_row_key_vector;
}
```

​		在实现缓存排序功能之前，必须先明白这一功能的合理性。不妨思考一下，既然可以在**数据库**中排序，为什么还要把排序功能放在缓存中实现呢？这里简单总结了两个原因：首先，排序会增加数据库的负载，难以支撑高并发的应用；其次，在缓存中排序不会遇到表锁定的问题。Redis恰好提供了排序功能，使我们可以方便地实现缓存排序。

​        Redis中用于实现排序功能的是SORT命令。该命令提供了多种参数，可以对列表，集合和有序集合进行排序。SORT命令格式如下：

```
SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination]
```

​		BY参数用于指定排序字段，功能类似于SQL中的order by。对于列表和集合而言，仅按照它们的值进行排序往往没有实际意义。以函数Cache2Hash返回的集合为例（实际上返回的是集合键），该集合中存储的是一系列完整的哈希键，只按照这些键进行排序，结果无非是按照数字或字典顺序排列，其用处显然不大。这是因为真正存储行数据的是哈希结构本身，而非哈希键。假设集合键为"resultset.hash:123456"，集合中每个哈希键对应的哈希结构中都有一个名为“timestamp”的字段，现在要把集合中的所有哈希键按照timestamp字段进行排序，这时，只需执行以下命令：

```
SORT resultset.hash:123456 BY *->timestamp
```

 		从上例可以看出，BY的真正威力在于它可以让SORT命令按照一个指定的外部键的外部字段进行排序。SORT用集合resultset.hash:123456中的每个值（即每个哈希键）替换BY参数后的第一个“*”，并依据“->”后面给出的字段获取其值，最后根据这些字段值对哈希键进行排序。

​        LIMIT参数用于限制排序以后返回元素的数量，功能类似于SQL中的limit。该参数接受另外两个参数，即offset和count，LIMIT offset count表示跳过前offset个元素，返回之后的连续count个元素。可见，LIMIT参数可以用于实现分页功能。

​        GET参数用于返回指定的字段值。以集合resultset.hash:123456为例，使用BY参数对集合中的所有哈希键按照哈希结构中的timestamp字段排序后，SORT命令返回所有排序之后的哈希键。如果某个请求需要不是键而是某些字段值，这时就要使用GET参数，使SORT命令返回指定字段值。假设除timestamp字段以外，集合中每个哈希键对应的哈希结构中还有一个名为“id”的字段，通过以下命令可以使SORT返回按照timestamp排序以后的每个哈希键对应的哈希结构中的timestamp和id值：

```
SORT resultset.hash:123456 BY *->timestamp GET *->timestamp GET *->id
```

​		SORT用集合resultset.hash:123456中的每个值（即每个哈希键）替换GET参数之后的第一个“*”，并将其作为返回值。值得注意的是，利用GET #能够得到集合中的哈希键本身。

​        ASC和DESC参数用于指定排序顺序（默认为ASC，即从低到高），ALPHA参数用于按照字典顺序排列非数字元素。

​        STORE参数用于将SORT命令的返回值，即排序结果存入一个指定的列表。加上STORE参数后，SORT命令的返回值就变为排序结果的个数。

​        下面的代码实现了按照哈希的某个字段对集合中的哈希键排序，并将结果存入列表的过程：

```
// 该函数对集合中的所有HASH键进行排序，排序依据是HASH键所对应的HASH中的某个字段，
// 排序结果被存入一个LIST结构，LIST键应当包含结果集标识符和排序字段标识符，
// 形如“sorted:123456:1234”
string SortHash(sql::Connection *mysql_connection,
                redisContext *redis_connection, 
                const string &resultset_id, 
                const string &sort_field, 
                int offset, int count, int order, int ttl) {
  // 只考虑存储HASH键的SET
  string redis_row_set_key = "resultset.hash:" + resultset_id;
  redisReply *reply;
  // 检测SET是否存在
  reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                               "EXISTS %s",
                                               redis_row_set_key.c_str()));
  if (reply->integer == 0) {
    freeReplyObject(reply);
    throw runtime_error("FAILURE - no resultsets");
  } else {
    freeReplyObject(reply);
  }
  string field_md5 = md5(sort_field);  // 利用MD5排除排序字段中空格造成的影响 
  // 将排序结果存入该LIST
  string redis_sorted_list_key = "sorted:" + resultset_id + ":" + field_md5;
  string by("*->" + sort_field);  //确定排序字段
  string ord = (order == 1) ? "ASC" : "DESC";  //order==1时按照升序排列；否则为降序
  stringstream ofsstream, cntstream;
  ofsstream << offset;
  cntstream << count;
  // 执行排序命令，并把排序结果存入LIST
  reply = static_cast<redisReply*>(redisCommand(
                                      redis_connection, 
                                      "SORT %s BY %s LIMIT %s %s GET %s ALPHA STORE %s",
                                      redis_row_set_key.c_str(), 
                                      by.c_str(), 
                                      ofsstream.str().c_str(), 
                                      cntstream.str().c_str(), 
                                      "#", 
                                      redis_sorted_list_key.c_str()));
  freeReplyObject(reply);
  stringstream ttlstream;
  ttlstream << ttl;
  // 设置LIST的过期时间
  reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                               "EXPIRE %s %s",
			                       redis_sorted_list_key.c_str(), 
                                               ttlstream.str().c_str()));
  freeReplyObject(reply);
  return redis_sorted_list_key;  // 返回LIST键，以便于其他函数获取该LIST中的内容
```

​		显然，对结果集中的哈希键进行排序要比对字符串键排序更加直观和方便。借助于排序函数，可以方便地实现在Redis中查询排序后的结果集，代码如下：

```
// 该函数根据sql语句和排序参数，在Redis中查询相应的结果集并进行排序，最后返回
// 排序之后的HASH键
vector<string> GetSortedCache(sql::Connection *mysql_connection,
                              redisContext *redis_connection,
                              const string &sql, const string &sort_field, 
                              int offset, int count, int order, int ttl) {
  vector<string> redis_row_key_vector;
  redisReply *reply;
  string resultset_id = md5(sql);  // 结果集标识符
  string field_md5 = md5(sort_field);  // 排序字段标识符
  // 尝试获取LIST中的所有HASH键
  string redis_sorted_list_key = "sorted:" + resultset_id + ":" + field_md5;
  // 尝试获取LIST中的所有HASH键
  reply = static_cast<redisReply*>(redisCommand(redis_connection, 
                                               "LRANGE %s %s %s",
                                               redis_sorted_list_key.c_str(), 
                                               "0", 
                                               "-1"));
  if (reply->type == REDIS_REPLY_ARRAY) {
    // 如果LIST不存在，调用Cache2Hash函数从Mysql中拉取数据到Redis，然后调用SortHash函数
    // 对结果集进行排序并将排序后的HASH键存入LIST
    if (reply->elements == 0) { 
      freeReplyObject(reply);
      sql::Statement *stmt = mysql_connection->createStatement();
      sql::ResultSet *resultset = stmt->executeQuery(sql);
      Cache2Hash(mysql_connection, redis_connection, resultset, 
                 resultset_id, ttl);
      redis_sorted_list_key = SortHash(mysql_connection, redis_connection, 
                                       resultset_id, sort_field, offset, 
                                       count, order, ttl);
      // 再次尝试获取LIST中的所有HASH键
      reply = static_cast<redisReply*>(redisCommand(
                                          redis_connection, 
                                          "LRANGE %s %s %s",
                                          redis_sorted_list_key.c_str(), 
                                          "0", 
                                          "-1"));
      delete resultset;
      delete stmt;
    }
    // 将LIST中的所有HASH键存入redis_row_key_vector中
    string redis_row_key;
    for (int i = 0; i < reply->elements; ++i) {
      redis_row_key = reply->element[i]->str;
      redis_row_key_vector.push_back(redis_row_key);
    }
    freeReplyObject(reply);
  } else {
    freeReplyObject(reply);
    throw runtime_error("FAILURE - LRANGE error");
  }
  return redis_row_key_vector;
}
```

 这样，在Redis中对结果集进行简单排序操作的功能就实现了。   

