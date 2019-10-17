---
layout: post
title: Mybatis相关
category: [Mybatis]
comments: false
---

* content
{:toc}

### 流式查询


#### MyMapper.java
```

interface MyMapper extends BaseMapper<MyResult> {

    public void test(ResultHandler<MyResult> resultHandler, @Param("date") String date);
}

```

#### MyMapper.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.test.mapper.MyMapper">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="com.test.dto.MyResult">
        <id column="xx" property="xx" />
        <result column="xxx" property="xxx" />
    </resultMap>

    <select id="test" fetchSize="300" resultSetType="FORWARD_ONLY" resultType="com.test.dto.MyResult">
        SELECT * FROM tb_test WHERE date >= #{date} ORDER BY date DESC
    </select>

</mapper>

```

#### MyService.java
```

@Service
public class MyServiceImpl extends ServiceImpl<MyMapper, MyResult> implements MyService{
    
    @Override
	public void test(String date) {
        this.baseMapper.test(new ResultHandler<MyResult>() {

			@Override
			public void handleResult(ResultContext<? extends MyResult> resultContext) {
                MyResult myResult = resultContext.getResultObject();
                ....
            }
        },date);
    }

}
```




