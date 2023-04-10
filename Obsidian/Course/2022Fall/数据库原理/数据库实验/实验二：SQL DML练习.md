# MySQL AUTO_INCREMENT：主键自增长
当主键定义为自增长后，这个主键的值就不再需要用户输入数据了，而由数据库系统根据定义自动赋值。每增加一条记录，主键会自动以相同的步长进行增长。
-   默认情况下，AUTO_INCREMENT 的初始值是 1，每新增一条记录，字段值自动加 1。
-  一个表中只能有一个字段使用 AUTO_INCREMENT 约束，且该字段必须有唯一索引，以避免序号重复（即为主键或主键的一部分）。
-  AUTO_INCREMENT 约束的字段必须具备 NOT NULL 属性。
-  AUTO_INCREMENT 约束的字段只能是整数类型（TINYINT、SMALLINT、INT、BIGINT 等）。
-  AUTO_INCREMENT 约束字段的最大值受该字段的数据类型约束，如果达到上限，AUTO_INCREMENT 就会失效。
# MySQL插入行
![[Pasted image 20221018143356.png]]
# 条件查询
如果有判断某一属性是否等于最大值的需求，查询到最大值后，尽管查询的结果中可能仅有一个元素，但子查询的结果也是一个集合（或table），不能将其看作一个元素。
因此应该使用IN来判断元素是否在子查询中，而不能使用=将元素与集合进行比较。
# IN的用法
IN后面需要加上一个集合或者一个子查询，而不能直接是一个table或table中的某一列，比如IN(Students.no)是非法的，而IN(SELECT no FROM Students)就是合法的
