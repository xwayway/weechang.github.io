---
title: 让你的Spring Data Jpa 像Mybatis一样灵活
tags: [Jpa, 动态查询]
date: 2018-11-30 11:31:03
categories: 日常记录
---

> 为了项目快速迭代，所以一直对spring data jpa 挺有好感的。但是，一直感觉jpa的查询方式不太灵活，特别是动态查询，及动态更新。但是最近在写快速开发框架，为了解决这个问题，在网上找到了相关资料，特此记录一下。

<!-- more -->

# 一、关于查询

## 1.1 古老的查询方式

一直以来只知道spring data jpa 可以免写sql。比如说我们要根据上级Id 查询所属的子信息。可以这样写

```Java

public interface DeptDao extends JpaRepository<T, Serializable> {

    List<Dept> findAllByParentId(Long parentId);
}
```

## 1.2 复杂的查询方式

这样的代码看似达到了我们的要求，但是如果我要加一个name作为查询条件呢？之前在网上上看到了类似这样的查询方式

```Java

@Override
public Page<Courier> pageQuery(Courier model, Pageable pageable) {

    //封装查询对象Specification
    Specification<Courier> example = new Specification<Courier>() {

        @Override
        public Predicate toPredicate(Root<Courier> root, CriteriaQuery<?> query, CriteriaBuilder cb) {

            //获取客户端查询条件
            String company = model.getCompany();
            String courierNum = model.getCourierNum();
            String type = model.getType();

            //定义集合来确定Predicate[] 的长度，因为CriteriaBuilder的or方法需要传入的是断言数组
            List<Predicate> predicates = new ArrayList<>();

            //对客户端查询条件进行判断,并封装Predicate断言对象
            if (StringUtils.isNotBlank(company)) {
                //root.get("company")获取字段名
                //company客户端请求的字段值
                //as(String.class)指定该字段的类型
                Predicate predicate = cb.equal(root.get("company").as(String.class), company);
                predicates.add(predicate);
            }
            if (StringUtils.isNotBlank(courierNum)) {
                Predicate predicate = cb.equal(root.get("courierNum").as(String.class), courierNum);
                predicates.add(predicate);
            }
            if (StringUtils.isNotBlank(type)) {
                Predicate predicate = cb.equal(root.get("type").as(String.class), type);
                predicates.add(predicate);
            }

            //判断结合中是否有数据
            if (predicates.size() == 0) {
                return null;
            }

            //将集合转化为CriteriaBuilder所需要的Predicate[]
            Predicate[] predicateArr = new Predicate[predicates.size()];
            predicateArr = predicates.toArray(predicateArr);

            // 返回所有获取的条件： 条件 or 条件 or 条件 or 条件
            return cb.or(predicateArr);
        }
    };



    //调用Dao方法进行条件查询
    Page<Courier> page = courierDao.findAll(example, pageable);
    return page;
}
```

## 1.3 优雅的查询方式

这TM的什么鬼啊，看都不想看，太复杂了。这TM一个查询比业务逻辑的代码都还多，要你有何用。于是乎，一直很嫌弃这种SB的写法。我坚信有更简单的实现方式，但是一直以来因为懒，没有去寻找解决方案。最近终于找到了如下解决方案。

首先还是Dao方法。
```Java
public interface DeptDao extends JpaRepository<T, Serializable> {

}
```

然后业务调用

```Java
@AutoWired
private DeptDao deptDao;

...

ExampleMatcher matcher = ExampleMatcher.matching()
                .withMatcher("name" ,ExampleMatcher.GenericPropertyMatchers.startsWith()) // 模糊查询匹配开头，即{name}%
                .withMatcher("remark" ,ExampleMatcher.GenericPropertyMatchers.contains()) // 全部模糊查询，即%{remark}%
                .withIgnorePaths("orderNum"); //忽略字段，即不管 orderNum 是什么值都不加入查询条件
                
baseDao.findAll(Example.of(param, matcher), pageable);

```

使用Example 查询，会自动忽略为NULL的字段，和mybatis的查询几乎一直。

我觉得，这才是我想要的查询方式。就是这么优雅。

