# 正在进行
- {{query (and (or [[NOW]] [[DOING]]) (not (page [[NOW]])) (not (page [[DOING]]) ) ) }}
  query-table:: false
-
- # 待办
- {{query (and (or [[LATER]] [[TODO]]) (not (page [[LATER]])) (not (page [[TODO]]) ) ) }}
- # 完成
- {{query (and (or [[DONE]] [[DONE]]) (not (page [[DONE]])) (not (page [[DONE]]) ) ) }}