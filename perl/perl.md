# Perl 基础语法

### 定义哈希
``` Perl
my %book = (
    'name' => 'RAID6',
    'price' => '12');
```
### 引用与解引用
``` Perl
my $book_ref = \%book;
my %uref = %{$book_ref};
```

### 直接执行命令
```perl
perl -e "print $aaa"
```
