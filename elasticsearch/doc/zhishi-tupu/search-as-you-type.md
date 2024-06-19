```
https://blog.csdn.net/UbuntuTouch/article/details/115653054


search_as_you_type 字段类型是一个类似 text 的字段，经过优化，可以为提供按需输入完成情况的查询提供开箱即用的支持。 它创建了一系列子字段，这些子字段被分析以索引可被部分与整个索引文本值匹配的查询有效匹配的术语。 支持前缀完成（即，匹配项从输入的开头开始）和中缀完成（即，匹配项在输入中的任意位置）。

将这种类型的字段添加到 mapping 时

PUT my-index-000001
{
  "mappings": {
    "properties": {
      "my_field": {
        "type": "search_as_you_type"
      }
    }
  }
}
这将创建以下字段：

my_field	按照 mapping 中的配置进行分析。 如果未配置分析器，则使用索引的默认分词器
my_field._2gram	用大小为 2 的 shingle token filter  分词器对 ny_field 进行分词
my_field._3gram	用大小为 3 的 shingle token filter  分词器对 ny_field 进行分词
my_field._index_prefix	用 edge ngram token filter 包装 my_field._3gram 的分词器



```