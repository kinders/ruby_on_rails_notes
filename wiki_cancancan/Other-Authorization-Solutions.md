There are [[many authorization solutions|http://steffenbartsch.com/blog/2008/08/rails-authorization-plugins/]] available, and it is important to find one which best meets the application requirements.
除了cancan，还有许多可用的授权方案，找到一个合适你的程序需求的方案，这很重要。
I've tried to keep CanCan minimal yet extendable so it can be used in many situations, but there are times it doesn't fit the best.
我已经在努力让CanCan变小、并且具有可扩展性，让它能用于许多情形；但又是它确实不是最合适的。

In earlier versions of CanCan there was no support for using the Ability logic in database queries, but 1.1 allows one to define permissions through a conditions hash which is usable in the database.
CanCan 的早期版本并不支持在数据库查询中使用能力逻辑，但1.1已经允许人们通过一个条件化的可用于数据库的散列来定义许可。
See [[Upgrading to 1.1]] for details on this new feature.
这个新特性详见《升级到1.1》。

If you find the conditions hash to be too limiting I encourage you to check out [[declarative_authorization|http://github.com/stffn/declarative_authorization]] which offers a sophisticated DSL for handling more complex permission scenarios.
如果发现条件散列太过有限，我鼓励你试试`declarative_authorization`，它提供了一个丰富的深度映射层来处理更复杂的许可场景。
This allows one to generate complex database queries based on the permissions but at the cost of a more complex DSL.
它允许人们生成复杂的基于许可的数据库查询，但也要耗费更复杂的DSL。

Also consider, if you have very unique authorization requirements, the best choice may be to write your own solution instead of trying to shoe-horn an existing plugin.
也要考虑到，如果你的授权需要非常独特，最好的选择可能是写一个自己的方案，而不是适应某种存在的插件。

*If you find a case where another tool meets a specific requirement better than CanCan, please edit this wiki page and add it.*
如果你找到哪种工具比cancan更能满足特定需要，请编辑这个维基页面，将它添加进来。
