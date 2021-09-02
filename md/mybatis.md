### mybatis

- jdbc

  1.  影响行数设置，默认操作mysql时返回的是查询的行数，而不是受影响的行数，可以修改useAffectedRows设置，例如

     ```yaml
     spring:
       datasource:
         url: jdbc:mysql://175.27.250.143:3306/dbName?useAffectedRows=true
         username: xxxxx
         password: xxxxx
     ```

     