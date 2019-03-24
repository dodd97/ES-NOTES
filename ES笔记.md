### 简单指令集
1. 快速检查集群健康情况： `GET /_cat/health?v`
2. 快速查看索引： `GET /_cat/indices?v`
3. 创建索引： `PUT /test_index?v`
4. 删除索引： `DELETE /test_index?v`

### 商品的基本CRUD操作
1. 新增商品。新建文档，建立索引： `PUT /index/type/id`
	eg:	
	``` 
	PUT /ecommerce/proudct/1
	{
		"name":"gaolujieyagao",
		"price":30",
		...
		"tages":["meibai","fangzhu"]
	}
	```
2. 更新：
  - `PUT /index/type/id` :本质是覆盖，更新某个条目的时候需要带上所有的field。
  - `POST /index/type/id_update` :实际意义上的更新。eg: 
	  ``` 
	POST /ecommerce/proudct/1_update
	{
  	"doc":{
  		"name":"gaolujieyagaojiaqiangban"
  		}
	}
  	```

3. 删除商品： `DELETE /index/type/id`

### 商品搜索指令
1. query string search  
	eg: 查询全部商品：  `GET /index/type/_search`   
	eg：查询全部名字中有”yagao“的商品并且按价格降序排序   
	`GET /ecommerce/product/_search?q=name:yagao&sort=price:dec`

2. query DSL  
  eg: 查询全部商品：  
	``` 
	GET /ecommerce/product/_search
	{
		"query":{
			"match_all":{}
		}
	}
	```
	eg：查询全部名字中有”yagao“的商品并且按价格降序排序
	``` 
	GET /ecommerce/product/_search
	{
		"query": {
			"match": {
				"name":"yagao"
			}
		},
		"sort": [{
			"price": {
				"order": "desc"
			}
		}]
	}
	```
	eg:分页查询商品
	``` 
	GET /ecommerce/product/_search
	{
		"query": {
			"match_all":{}
		},
		"from":1,	//查询哪一页的商品，0为第一条
		"size":1	//查询哪页
	}
	```
	eg:查询所有商品的名称和价格
	``` 
	GET /ecommerce/product/_search
	{
		"query": {
			"match_all":{}
		},
		"_source": ["name","price"]
	}
	```

3. query filter //筛选
	eg:筛选名字内有“yagao”且价格大于25的商品
	``` 
	GET /ecommerce/product/_search
	{
		"query":{
	    	"bool":{
	      		"must":{
	        		"match":{
	          			"name": "yagao"
	       	 		}
	      		},
	      		"filter":{
	        		"range":{
	          			"price":{
	            			"gt":25
	          			}
	        		}
	      		}
	    	}
	  	}
  	}
	```

4. full-text search	//全文检索
	eg:搜索生产商带有"yagao"__或者__"producer"的商品
	``` 
	GET /ecommerce/product/_search
	{
	  	"query":{
			"match:{
				"producer":"yagao producer"
			}
		}
  	}//"yagao"和"producer"会被拆开分别在倒排索引中比对，如果某商品满足多个词会有更高的"scores"
	```

5. phrase search	//短语搜索
	eg:搜索生产商带有整个"yagao producer"短语的商品
	``` 
	GET /ecommerce/product/_search
	{
	  	"query":{
			"match_phrase:{
				"producer":"yagao producer"
			}
		}
  	}//必须是包含"yagao producer"整个短语的商品，不可拆分
	```

6. highlight search	//高亮搜索结果
	eg:搜索生产商带有整个"yagao producer"短语的商品并加亮搜索结果中的"producer"
	``` 
	GET /ecommerce/product/_search
	{
	  	"query":{
			"match_phrase:{
				"producer":"yagao producer"
			}
		},
		"highlight":{
			"fields"：{
				"producer":{}
			}
		}
  	}
	```

### 商品管理指令
eg: 
1. 计算每个tag下的商品数量
	``` 
   	GET /ecommerce/product/_search
	{
		"aggs":{
    		"group_by_tags":{
      			"terms":{
        			"field":"tags"
      			}
    		}	
  		}
	}
	```
2. 聚合分析：先分组，在算每组的平均值，计算每个tag下商品的平均价格
	``` 
	GET /ecommerce/product/_search
	{
  		"size":0,	//简介化输出界面
  		"aggs":{
    		"group_by_tags":{
      			"terms":{
        			"field":"tags"
      			},
      			"aggs":{
        			"avg_price":{
          				"avg":{
            				"field": "price"
          				}
        			}
      			}
    		}
  		}
	}
	```

3. 
