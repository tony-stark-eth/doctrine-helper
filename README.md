<img src="./title-white.png" width="auto"/>

English | [Chinese](README_zh.md)

This tool provides an alternative to the `php bin/console doctrine:mapping:import App\\Entity attribute --path=src/Entity` command in Symfony 7, as the `doctrine:mapping:import` command has been removed in the latest versions. It allows for importing entity mappings from existing databases into Symfony applications.

## Supported Databases

- MySQL
- PostgreSQL
- SQLServer
- Oracle
- Sqlite3

## Supported Field Types

### MySQL
- Does not support `bit` types
- When supporting `enum` types, you need to set [mapping_types](https://symfony.com/doc/current/doctrine/dbal.html#registering-custom-mapping-types-in-the-schematool)
- Converts similar types (e.g., `double` to `float`)

### PostgreSQL
- Supports numeric, string, date, currency, JSON, UUID, binary, and boolean types
- Does not support complex or custom types

### SQLServer
- Supports exact numeric, approximate numeric, date/time, string (char, varchar), Unicode string (nchar, nvarchar), binary string (binary, varbinary), and other data types
- Does not support deprecated or complex data types

### Oracle
- Supports various data types such as NUMBER, FLOAT, CHAR, VARCHAR2, DATE, TIMESTAMP, RAW, CLOB, BLOB, etc.
- Does not support other data types

### Sqlite
- Supports integer, real, float, double, decimal, varchar, blob, text, date, datetime, and boolean types

## Install
```shell
composer require siburuxue/doctrine-helper
```

## Synchronize database tables to the project
```shell
php bin/console doctrine-helper:mapping:import App\\Entity attribute --path=src/Entity --ucfirst=true --table=dict,log --without-table-prefix=eq_
php bin/console doctrine-helper:mapping:import --ucfirst=true
```

## Command Line Options

- `namespace`: The namespace for the Entity class (default: `App\Entity`)
- `type`: Type of database field description (attribute, xml, yaml, php) (default: `attribute`, currently only support `attribute`)
- `--path`: Path to store Entity class files (default: `src/Entity`)
- `--ucfirst=true`: Generate Symfony 6 style Entities (private properties in camelCase) for seamless code migration (default: Symfony 7 style with underscored private properties)
- `--table`: Import specific tables to generate corresponding Entity and Repository classes
- `--without-table-prefix`: Ignore table prefix when generating Entities
- `--schema`: when connecting to a postgresql database, synchronize the tables of the specified schema (postgresql only)

In this article, we'll use MySQL as an example to demonstrate the generation result.

Assuming your database contains a table called "test," which includes almost all MySQL data types and has unique indexes, composite indexes, and regular indexes:

```sql
create table test
(
    id     int                                     not null,
    int_1  int                 default 1           not null comment 'int',
    int_2  smallint            default 2           null comment 'smallint',
    int_3  int                 default 3           null comment 'tinyint',
    int_4  mediumint           default 4           null comment 'mediumint',
    int_5  bigint                                  not null comment 'bigint'
        primary key,
    int_6  double              default 6           null comment 'float',
    int_7  double              default 7           null comment 'double',
    int_8  decimal             default 8           null comment 'decimal',
    date_1 date                default (curdate()) null comment 'date',
    date_2 time                default (curtime()) null comment 'time',
    date_3 datetime            default (now())     null comment 'datetime',
    date_4 timestamp           default (now())     null comment 'timestamp',
    date_5 year                default (now())     null comment 'year',
    str_1  char                default 'a'         null comment 'char',
    str_2  varchar(255)        default 'b'         null comment 'varchar(255)',
    str_3  varbinary(1)        default 0x63        null comment 'binary',
    str_4  varbinary(1)        default 0x64        null comment 'varbinary(1)',
    str_8  set ('a', 'b', 'c') default 'a,b'       null comment 'set',
    json_1 json                                    null comment 'json',
    bool_1 int                 default 0           null comment 'bool',
    constraint I_int_2
        unique (int_2) comment '唯一索引'
)
    comment '测试表';

create index I_int_1
    on test (int_1)
    comment '普通索引';

create index I_int_3
    on test (int_3, int_4, int_5)
    comment '联合索引';

create index I_int_4
    on test (int_6);
```

To generate the corresponding Entity and Repository class files using the command:

```shell
php bin/console doctrine-helper:mapping:import --ucfirst=true --table=test
```

The newly created Test entity class will look like this:

```php
// src/Entity/Test.php
namespace App\Entity;

use App\Repository\TestRepository;
use Doctrine\DBAL\Types\Types;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Table(name: 'test')]
#[ORM\UniqueConstraint(name: 'I_int_2', columns: ['int_2'])]
#[ORM\Index(name: 'I_int_1', columns: ['int_1'])]
#[ORM\Index(name: 'I_int_3', columns: ['int_3', 'int_4', 'int_5'])]
#[ORM\Index(name: 'I_int_4', columns: ['int_6'])]
#[ORM\Entity(repositoryClass: TestRepository::class)]
class Test
{
    #[ORM\Column(name: "int_5", type: Types::BIGINT, options: ["comment" => "bigint"])]
    #[ORM\Id]
    #[ORM\GeneratedValue(strategy: "NONE")]
    private ?string $int5 = null;

    #[ORM\Column(name: "id")]
    private ?int $id = null;

    #[ORM\Column(name: "int_1", options: ["comment" => "int", "default" => 1])]
    private ?int $int1 = 1;

    #[ORM\Column(name: "int_2", type: Types::SMALLINT, nullable: true, options: ["comment" => "smallint", "default" => 2])]
    private ?int $int2 = 2;

    #[ORM\Column(name: "int_3", nullable: true, options: ["comment" => "tinyint", "default" => 3])]
    private ?int $int3 = 3;

    #[ORM\Column(name: "int_4", nullable: true, options: ["comment" => "mediumint", "default" => 4])]
    private ?int $int4 = 4;

    #[ORM\Column(name: "int_6", nullable: true, options: ["comment" => "float", "default" => 6])]
    private ?float $int6 = 6;

    #[ORM\Column(name: "int_7", nullable: true, options: ["comment" => "double", "default" => 7])]
    private ?float $int7 = 7;

    #[ORM\Column(name: "int_8", type: Types::DECIMAL, precision: 10, scale: 0, nullable: true, options: ["comment" => "decimal", "default" => 8])]
    private ?string $int8 = '8';

    #[ORM\Column(name: "date_1", type: Types::DATE_MUTABLE, nullable: true, options: ["comment" => "date", "default" => 'curdate()'])]
    private ?\DateTimeInterface $date1 = null;

    #[ORM\Column(name: "date_2", type: Types::TIME_MUTABLE, nullable: true, options: ["comment" => "time", "default" => 'curtime()'])]
    private ?\DateTimeInterface $date2 = null;

    #[ORM\Column(name: "date_3", type: Types::DATETIME_MUTABLE, nullable: true, options: ["comment" => "datetime", "default" => 'now()'])]
    private ?\DateTimeInterface $date3 = null;

    #[ORM\Column(name: "date_4", type: Types::DATETIME_MUTABLE, nullable: true, options: ["comment" => "timestamp", "default" => 'now()'])]
    private ?\DateTimeInterface $date4 = null;

    #[ORM\Column(name: "date_5", type: Types::DATETIME_MUTABLE, nullable: true, options: ["comment" => "year", "default" => 'now()'])]
    private ?\DateTimeInterface $date5 = null;

    #[ORM\Column(name: "str_1", length: 1, nullable: true, options: ["comment" => "char", "fixed" => true, "default" => 'a'])]
    private ?string $str1 = 'a';

    #[ORM\Column(name: "str_2", length: 255, nullable: true, options: ["comment" => "varchar(255)", "default" => 'b'])]
    private ?string $str2 = 'b';

    #[ORM\Column(name: "str_3", type: Types::BINARY, length: 1, nullable: true, options: ["comment" => "binary", "default" => '0x63'])]
    private $str3 = 0x63;

    #[ORM\Column(name: "str_4", type: Types::BINARY, length: 1, nullable: true, options: ["comment" => "varbinary(1)", "default" => '0x64'])]
    private $str4 = 0x64;

    #[ORM\Column(name: "str_8", type: Types::SIMPLE_ARRAY, nullable: true, options: ["comment" => "set", "default" => 'a,b'])]
    private ?array $str8 = ["a","b"];

    #[ORM\Column(name: "json_1", nullable: true, options: ["comment" => "json"])]
    private ?array $json1 = null;

    #[ORM\Column(name: "bool_1", nullable: true, options: ["comment" => "bool", "default" => 0])]
    private ?int $bool1 = 0;

    // ...getter and setter
}
```

Feel free to explore and utilize this tool for managing Doctrine mappings in your Symfony applications!

## Contact Us(QQ)
<img src="./QQ.jpg" width="400px"/>
