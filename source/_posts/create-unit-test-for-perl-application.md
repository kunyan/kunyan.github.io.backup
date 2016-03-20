title: Create Unit Test for Perl application
date: 2016-03-20 22:57:13
tags:
---

Create Unit Test for Perl application
---

### 1. Install Test::More

###### Fedora
```shell
sudo dnf install perl-Test-Simple
```

###### RHEL
```shell
sudo yum install perl-Test-Simple
```

###### Ubuntu
```shell
sudo apt-get install libtest-simple-perl
```

### 2. Code Rule

##### Naming

The test file's name start with "Test" and end in ".t", it should saved in "t/" directory.

The recommend naming is "Test" + The Module be tested name + ".t"

Example:
```
TestVendorProduct.t
```

##### Indentation
4 spaces. no Tabs. Make sure your editor has been setup ready before coding.

### 3. Create the first test file

```perl
#!/usr/bin/env perl
use strict;
use warnings;
use Test::More tests => 2;

my $got = 'Hello World';
my $expected = 'Hello World';
is  ( $got, $expected, 'Say Hello to World' );

my $got2 = 'Hello Moon';
my $expected2 = 'Hello Mars';
is  ( $got2, $expected2, 'Say Hello to Mars' );

```
Output:
```
1..2
ok 1 - Say Hello to World
not ok 2 - Say Hello to Mars
#   Failed test 'Say Hello to Mars'
#   at /home/kyan/perl-unittest/t/TestUnitCase.t line 12.
#          got: 'Hello Moon'
#     expected: 'Hello Mars'
# Looks like you failed 1 test of 2.
```

##### Test Plan

When you create a test file for testing a module, you would know how many functions will be tested in this test file. there have 2 ways to define test numbers
```perl
use Test::More tests => 2; # 2 is the number of tests
```
or

```perl
use Test::More;

# Testing code ....

done_testing(2);
```

if you don't know how many tests you're going to run, you can just use

```perl
done_testing();
```

##### Assertions

1. ok
```perl
ok($got eq $expected, $test_name);
```

2. is
```perl
 is  ( $got, $expected, $test_name );
```

3. isnt
```perl
isnt( $got, $expected, $test_name );
```
4. like
```perl
like( $got, qr/expected/, $test_name );
```

5. unlike
```perl
unlike( $got, qr/expected/, $test_name );
```
6. cmp_ok
```perl
cmp_ok( $got, $op, $expected, $test_name );
# ok( $got eq $expected );
cmp_ok( $got, 'eq', $expected, 'this eq that' );
```

7. can_ok
```perl
can_ok($module, @methods);
can_ok($object, @methods);
```

8. isa_ok
```perl
isa_ok($object,   $class, $object_name);
isa_ok($subclass, $class, $object_name);
isa_ok($ref,      $type,  $ref_name);
```

9. new_ok
```perl
my $obj = new_ok( $class );
my $obj = new_ok( $class => \@args );
my $obj = new_ok( $class => \@args, $object_name );
```

10. subtest
```perl
use Test::More tests => 1;

  subtest 'An example subtest' => sub {
      plan tests => 2;

      pass("This is a subtest");
      pass("So is this");
  };
```

11. pass
```perl
pass($test_name);
```

12. fail
```perl
fail($test_name);
```

13. require_ok
```perl
require_ok($module);
require_ok($file);
```

14. use_ok
```perl
BEGIN { use_ok($module); }
BEGIN { use_ok($module, @imports); }
```

15. is_deeply
```perl
is_deeply( $got, $expected, $test_name );
```

16. eq_array
```perl
my $is_eq = eq_array(\@got, \@expected);
```

17. eq_hash
```perl
my $is_eq = eq_hash(\%got, \%expected);
```

18. eq_set
```perl
my $is_eq = eq_set(\@got, \@expected);
```

##### Diagnostics

1. diag
```perl
diag(@diagnostic_message);
```

2. note
```perl
note(@diagnostic_message);
```

3. explain
```perl
my @dump = explain @diagnostic_message;
```

##### Blocks

1. SKIP
```perl
SKIP: {
    skip $why, $how_many if $condition;

    ...normal testing code goes here...
}
```


2. TODO
```perl
TODO: {
    local $TODO = $why if $condition;

    ...normal testing code goes here...
}
```

### 4. Continuous integration with Jenkins

#### Perl Application Environment Configuration
If you want to running test, your have to setup your application configuration.
Sometimes you have a config file, it is different in each env(Dev/Staging/Prod), What should we do now ?

Here has a plugin could help you.

* [Config File Provider Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Config+File+Provider+Plugin)

After installed it, Goto System config -> Managed files, Add a new Config.
Then go to your job, "Provide Configuration files", put the file in "Target" which you want to place it.

#### Generate Tests Report

##### 1. Install TAP::Harness::Archive

###### Fedora
```shell
sudo dnf install perl-TAP-Harness-Archive
```

###### RHEL
```shell
sudo yum install perl-TAP-Harness-Archive
```

###### Ubuntu
```shell
sudo apt-get install libtap-harness-archive-perl
```

##### 2. Run prove to generate test reports
create a directory for outputs and execute prove command to generate test reports

```shell
mkdir -p output

prove -r t/ --archive output
```

You will found test results from output directory.

##### 3. Publish test reports on Jenkins

Install [TAP Plugin](https://wiki.jenkins-ci.org/display/JENKINS/TAP+Plugin)

Configure your job, Add a task after build, "Publish TAP Results",
Input the test file pattern like :
```
output/**/*.t
```

#### Generate Code Coverage

##### 1. Install Cover::Devel

###### Fedora
```shell
sudo dnf install perl-Cover-Devel
```

###### RHEL
```shell
sudo yum install perl-Cover-Devel
```

###### Ubuntu
```shell
sudo apt-get install libdevel-cover-perl
```

##### 2. Install Devel::Cover::Report::Clover

I didn't found any exists package, please use CPAN install it

```shell
cpan install Devel::Cover::Report::Clover
```

##### 3. Run cover to generate code coverage reports
```shell
cover --report clover
```

##### 4. Publish Code coverage reports on Jenkins

* Install [Clover Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Clover+Plugin)
* Install [HTML Publisher Plugin](https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin)

Configure your job, Add a task after build, "Publish Clover Coverage Report",
* Clover report directory : cover_db
* Clover report file name : clover.xml

Add a task after build, "Publish HTML reports",
* HTML directory to archive : cover_db
* Index page[s] : coverage.html
* Report title : Coverage Reports


The job Execute shell should be:

```shell
prove -r t/ --archive output
cover --report clover
```


### Known Bugs:
Clover Plugin has a bug, the "Coverage Report" link is 404, So I use HTML Publisher Plugin to instead of it.


### References
1. [Test More](http://search.cpan.org/~exodist/Test-Simple-1.001014/lib/Test/More.pm)
