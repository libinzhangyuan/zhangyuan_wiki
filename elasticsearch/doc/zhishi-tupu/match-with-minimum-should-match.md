```

{
    "query": {
        "match": {
            "product_shop_code": {
                "query": "9527 8848",
                "minimum_should_match": 2
            }
        }
    }
}



{
    "query": {
        "match": {
            "product_shop_code": {
                "query": "9527-8848",
                "minimum_should_match": 2
            }
        }
    }
}

```