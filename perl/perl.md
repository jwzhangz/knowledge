# Perl 基础语法

### 定义哈希

    my %book = (
        'name' => 'RAID6',
        'price' => '12');

### 引用与解引用
my $book_ref = \%book;
my %uref = %{$book_ref};
