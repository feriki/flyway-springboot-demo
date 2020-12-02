通过springboot集成flyway的样例。

基本命令： Migrate： Migrate是指把数据库Schema迁移到最新版本，是Flyway工作流的核心功能，Flyway在Migrate时会检查Metadata(元数据)表，如果不存在会创建Metadata表，Metadata表主要用于记录版本变更历史以及Checksum之类的。 Migrate时会扫描指定文件系统或Classpath下的Migrations(可以理解为数据库的版本脚本)，并且会逐一比对Metadata表中的已存在的版本记录，如果有未应用的Migrations，Flyway会获取这些Migrations并按次序Apply到数据库中，否则不需要做任何事情。另外，通常在应用程序启动时应默认执行Migrate操作，从而避免程序和数据库的不一致性。

Clean： Clean操作在开发和测试阶段是非常有用的，它能够帮助快速有效地更新和重新生成数据库表结构，但特别注意的是：不应在Production的数据库上使用！

Info： Info用于打印所有Migrations的详细和状态信息，其实也是通过Metadata表和Migrations完成的，下图很好地示意了Info打印出来的信息。 Info能够帮助快速定位当前的数据库版本，以及查看执行成功和失败的Migrations。

Validate： Validate是指验证已经Apply的Migrations是否有变更，Flyway是默认是开启验证的。 Validate原理是对比Metadata表与本地Migrations的Checksum值，如果值相同则验证通过，否则验证失败，从而可以防止对已经Apply到数据库的本地Migrations的无意修改。

Baseline： Baseline针对已经存在Schema结构的数据库的一种解决方案，即实现在非空数据库中新建Metadata表，并把Migrations应用到该数据库。 Baseline可以应用到特定的版本，这样在已有表结构的数据库中也可以实现添加Metadata表，从而利用Flyway进行新Migrations的管理了。

Repair： Repair操作能够修复Metadata表，该操作在Metadata表出现错误时是非常有用的。 Repair会修复Metadata表的错误，通常有两种用途：

移除失败的Migration记录，该问题只是针对不支持DDL事务的数据库。 重新调整已经应用的Migratons的Checksums值，比如：某个Migratinon已经被应用，但本地进行了修改，又期望重新应用并调整Checksum值，不过尽量不要这样操作，否则可能造成其它环境失败。

脚本文件命名规则 文件名由以下部分组成，除了使用默认配置外，某些部分还可自定义规则。 
prefix: 可配置，前缀标识，默认值V表示Versioned，R表示Repeatable。 version: 标识版本号，由一个或多个数字构成，数字之间的分隔符可用点.或下划线_。 separator: 可配置，用于分隔版本标识与描述信息，默认为两个下划线__。 description: 描述信息，文字之间可以用下划线或空格分隔。 suffix: 可配置，后续标识，默认为.sql。

##多数据源配置##

@Configuration
public class DataSourceConfig {
    
    //  一定要指定数据源
    @FlywayDataSource
    @Bean(name = "mysql")
    @ConfigurationProperties(prefix = "spring.datasource.druid.mysql")
    public DataSource mysql() {
        return new DruidDataSource();
    }

    @Bean(name = "mycat")
    @ConfigurationProperties(prefix = "spring.datasource.druid.mycat")
    public DataSource mycat() {
        return new DruidDataSource();
    }
}


