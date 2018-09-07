索引 =》 数据库
类型 =》 表
文档 =》 行记录

查看文档[https://blog.csdn.net/u010648555/article/details/81841188](https://blog.csdn.net/u010648555/article/details/81841188)

备注:
安装集群的时候,在修改了配置重新启动的时候一定要删除`elasticsearch`根目录的`data`文件, 否则会出现莫名的错误.

### 执行脚本
```sh
# 创建索引
curl -X POST "http://127.0.0.1:9200/gbl_v1"

curl -X PUT \
  http://10.255.73.106:9500/gbl \
  -H 'Content-Type: application/json' \
  -d '{
	"settings": {
		"number_of_shards": 5,
		"number_of_replicas": 1
	},
	"mappings": {
		"m": {
			"properties":{
				"did":{
					"type": "text"
				}
			}
		}
	}
}'

# 删除索引
curl -XDELETE localhost:9200/gbl_v1

# 查看mapping内容
curl -XGET "http://127.0.0.1:9200/gbl_v1/_mapping?pretty"
# {
#   "gbl_v1" : {
#     "mappings" : { }
#   }
# }

# 增加mapping
curl -XPOST "http://127.0.0.1:9200/gbl_v1/m/_mapping?pretty" -d '{
  "m": {
    "properties": {
        "age": {
            "type": "integer"
        },
        "name": {
            "type": "text"
        }
    }
  }
}'

# 再次查看
curl -X GET \
  http://10.255.73.106:9500/gx/m/_mapping \
  -H 'Content-Type: application/json' \

# 新增mapping字段
curl -X PUT \
  http://10.255.73.106:9500/test/_mapping/m \
  -H 'Content-Type: application/json' \
  -d '{
	"properties": {
	    "gender": {
	      "type": "integer"
	    }
	 }
}'


# 新增数据
curl -X POST \
  http://10.255.73.106:9500/gx/m \
  -H 'Content-Type: application/json' \
  -d '{
	"age": 2833,
	"name": "pjj"
}'

# 指定记录的插入的id
curl -X POST \
  http://10.255.73.106:9500/test/m/1 \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 4acef5fe-76fd-4f01-b20d-15dbe693f3a0' \
  -d '{
	"age": 18
}'

# 创建索引昵称
curl -X POST \
  http://10.255.73.106:9500/_aliases \
  -H 'Content-Type: application/json' \
  -d '{
	"actions": [
        {
        	"add": {
	            "alias": "gx",
            	"index": "test"
            }
        }
    ]
}'

# 修改索引的昵称
curl -X POST \
  http://10.255.73.106:9500/_aliases \
  -H 'Content-Type: application/json' \
  -d '{
    "actions": [
        { "remove": {
            "alias": "gx",
            "index": "test"
        }},
        { "add": {
            "alias": "gx",
            "index": "test2"
        }}
    ]
}'
# 直接查询某个id的记录
curl -X GET \
  http://10.255.73.106:9500/:index/:type/:id \
  -H 'Cache-Control: no-cache' \
  -H 'Postman-Token: 1cf1c853-cf1f-4f40-a6c2-8bfce83d5c5c'

# 给索引创建别名
curl -XPUT "http://192.168.99.1:9200/oldindex/_alias/alias_oldindex"

# 重置索引
curl -X POST \
  http://10.255.73.106:9500/_reindex \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 9df4d937-c5b4-4e6f-a998-79a3d8ee7b8b' \
  -d '{
	"size": 10000,
	"conflicts":  "proceed",
	"source": {
		"size": 20,
		"index": "gbl",
		"type": "m",
		"_source": ["*"]
	},
	"dest": {
		"index": "gbl_v2",
		"op_type": "create",
		"routing": "=routingvalue"
	}
}'

```


```json
{
  "query": {
    "match_phrase": { //绝对匹配
        "title": "Elastic"
    }
  }
}

{
  "query": {
    "multi_match": {
        "query": "Elastic",
        "fields": ["author", "title"]
    }
  }
}

{
  "query": {
    "query_string": {
      "query": "Elastic AND Python",
      // "query": "(Elastic AND Python) OR GOLANG",
      "fields": ["title", "author"]
    }
  }
}

{
  "query": {
    "term": {
      "word_count": 1000,
    }
  }
}


{
  "query": {
    "range": {
      "word_count": {
        "gte": 1000,
        "lte": 2000
      }
    }
  }
}

{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "word_count": 1000
        }
      }
    }
  }
}

{
  "query":{
    "bool": {
      "should": [
        "match": {
          "author": "xx"
        },
        {
          "match": {
            "name": ""
          }
        }
      ]
    }
  }
}

{
  "query":{
    "bool": {
      "must": [
        "match": {
          "author": "xx"
        },
        {
          "match": {
            "name": "'"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "word_count": 1000
          }
        }
      ]
    }
  }
}
```

