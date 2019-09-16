---
layout: posts
title:  "compiler 02 practicce"
date:   2019-09-16
categories: cs
---
컴파일러 입문 2장 연습문제.  
> 다음은 PL/0 문법이다. PL/0 언어를 이용하여 다음을 프로그래밍 하시오.
> 500 이하의 완전수(perfect number를 구하시오. 완전수란 자신을 제외한 약수의 합과 같은 양의 정수를 말한다. 
```
<DCL> <STATEMENT>.

<BLOCK>         -> <DCL> <STATEMENT>
<DCL>           -> <CONST_DCL> <VAR_DCL> <PROC_DCL>
<CONST_DCL>     -> CONST <CONST_DEF+> ; | e   // CONST x = 50; a = 10;
<VAR_DCL>       -> VAR <VAR_DEF+> ; | e       // VAR x, a;
<PROC_DCL>      -> <PROC_DCL> <PROC_DCL_1> | e  // PROCEDURE asd_function ; <BLOCK>

<PROC_DCL_1>    -> <PROC_HEAD> <BLOCK> 
<PROC_HEAD>     -> PROCEDURE <identifier> ;

<CONST_DEF+>    -> <CONST_DEF> | <CONST_DEF+> ; <CONST_DEF> // x = 50; a = 10
<CONST_DEF>     -> <identifier> = <number>

<VAR_DEF+>      -> <identifier> | <VAR_DEF+> , <identifier> // x, a

<STATEMENT>     -> <ASSIGNMENT_ST> | <CALL_ST> | <COMPOUND_ST> | <IF_ST> | <WHILE_ST> | e
<STATEMENT*>    -> <STATEMNET*> ; <STATEMENT> | <STAETMENT>
<ASSIGNMENT_ST> -> <identifer> := <EXP>
<CALL_ST>       -> CALL <identifier>
<COMPOUND_ST>   -> BEGIN <STATEMENT*> END
<IF_ST>         -> IF <CONDITION> THEN <STATEMENT>
<WHILE_ST>      -> WHILE <CONDITION> DO <STATEMENT>

<CONDITION>     -> <EXP> <CON_OPERATOR> <EXP>
<CON_OPERATOR>  -> = | <> | < | > | <= | >=
<EXP>           -> <EXP> + <TERM> | <EXP> - <TERM> | <TERM>
<TERM>          -> <TERM> * <FACTOR> | <TERM> / <FACTOR> | <FACTOR>
<FACTOR>        -> -<FACTOR> | <identifer> | <number> | (<EXP>)
```
아래는 
```
CONST MAX_NUMBER = 500;
VAR number, index, sum, perfect_number;

number := 1;
WHILE number <= MAX_NUMBER DO 
BEGIN
  sum := 0;
  index := 1;
  WHILE index <> number DO
  BEGIN
    IF number / index * index = number THEN
    BEGIN
      sum += index
    END;

    IF sum = number THEN
    BEGIN
      perfect_number = number
    END
  END;

  number := number + 1
END
.
```
