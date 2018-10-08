商品搜索：
1.全部商品写入到索引库：
从数据库查询所有商品，遍历全部商品，分别获取商品的id、title、卖点、价格、图片url、分类名称，分别向solr文档对象中添加域；
其中id添加域是必须的；
步骤：
1）创建文档对象org.apache.solr.common.SolrInputDocument；
2）向文档对象中添加域；
代码：
//创建文档对象
SolrInputDocument document = new SolrInputDocument();
//向文档对象中添加域
document.addField("id", searchItem.getId());
document.addField("item_title", searchItem.getTitle());
document.addField("item_sell_point", searchItem.getSell_point());
document.addField("item_price", searchItem.getPrice());
document.addField("item_image", searchItem.getImage());
document.addField("item_category_name", searchItem.getCategory_name());
//把文档对象写入索引库
solrServer.add(document);

2.实现关键词、分页搜索：
步骤：
1）创建org.apache.solr.client.solrj.SolrQuery对象；
2）设置查询条件：设置关键词查询域，查询页码，每页查询条数；
代码：
SolrQuery query = new SolrQuery();
//设置查询条件
query.setQuery(keyword);
//设置分页条件
if (page <=0 ) page =1;
query.setStart((page - 1) * rows);
query.setRows(rows);
//设置默认搜索域
query.set("df", "item_title");
//开启高亮显示
query.setHighlight(true);
query.addHighlightField("item_title");
query.setHighlightSimplePre("<em style=\"color:red\">");
query.setHighlightSimplePost("</em>");