```
cd work/serv/wiki
. run.sh

cd ../zhangyuan_wiki
. run.sh


# 启动es  - 另一个控制台
/home/test/es/elasticsearch-5.5.0/bin/elasticsearch
hosts:  es.cn   es
es:9200



# 启动kibana - 新开控制台
/home/test/es/kibana-5.5.0-linux-x86_64/bin/kibana
hosts:  kibana
kibana:5601


```











```



DELETE nbdpress


GET	nbdpress/_mapping/articles

GET nbdpress/articles/_search?q=Yes, they share their failure stories, publicly





PUT	/nbdpress
{
		"settings"	:	{
			"number_of_shards"	:	5,
			"number_of_replicas"	:	0
		},
		"mappings":	{
				"articles"	:	{
						"properties"	:	{
								"title"	:	{
								  "type"	:				"text",
									"analyzer":	"english"
								},
								"digest"	:	{
								  "type"	:				"text",
									"analyzer":	"english"
								},
								"content"	:	{
								  "type"	:				"text",
										"analyzer":	"english"
								},
								"tags"	:	{
										"type"	:				"text",
										"index" : false
								},
								"published_at"	:	{"type": "date"},
								"copyright": {"type" : "long"},
								"status" : {"type": "long"},
								"ori_source" : {"type": "text", "index": false}
						}
				}
		}
}



POST /nbdpress/articles/
{
  "title": "My first blog entry",
  "digest": "chengdu is good!",
  "content":  "Just trying this out...",
  "tags": ["chengdu", "food", "beauty"],
  "published_at":  "2014-01-01",
  "copyright": 1,
  "status": 1,
  "ori_source": "nbd"
}

GET nbdpress/articles/_search?q=chengdu


GET nbdpress/articles/query?explain
{
    "query": {
        "match_phrase": {
            "title": {
                "query": "first my",
                "slop": 2
            }
        }
    }
}



GET /nbdpress/articles/_search
{
    "query": {
        "match_phrase": {
            "title": {
                "query": "first my",
                "slop": 2
            }
        }
    }
}

```