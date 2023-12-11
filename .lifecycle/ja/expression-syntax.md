---
title: 式の構文
summary: TiDBにおける式の構文について学びます。
aliases: ['/docs/dev/expression-syntax/','/docs/dev/reference/sql/language-structure/expression-syntax/']
---

# 式の構文

式とは、1つ以上の値、演算子、または関数の組み合わせです。TiDBでは、式は主に`SELECT`文のさまざまな句で使用されており、Group by句、Where句、Having句、Join条件、ウィンドウ関数などがあります。また、一部のDDL文も式を使用しており、テーブルの作成時にデフォルト値、列、およびパーティションの規則の設定などがあります。

式は以下のタイプに分類されます。

- 識別子。参照については、[スキーマオブジェクト名](/schema-object-names.md)を参照してください。

- 述語、数値、文字列、日付式。これらのタイプの[リテラル値](/literal-values.md)も式です。

- 関数呼び出しおよびウィンドウ関数。参照については、[関数と演算子の概要](/functions-and-operators/functions-and-operators-overview.md)および[ウィンドウ関数](/functions-and-operators/window-functions.md)を参照してください。

- ParamMarker (`?`)、システム変数、ユーザ変数、CASE式。

以下の規則は、[parser.y](https://github.com/pingcap/parser/blob/master/parser.y) のTiDBパーサーの規則に基づいた式の構文です。以下の構文図のナビゲート可能なバージョンについては、[TiDB SQL構文図](https://pingcap.github.io/sqlgram/#Expression)を参照してください。

```ebnf+diagram
Expression ::=
    ( singleAtIdentifier assignmentEq | 'NOT' | Expression ( logOr | 'XOR' | logAnd ) ) Expression
|   'MATCH' '(' ColumnNameList ')' 'AGAINST' '(' BitExpr FulltextSearchModifierOpt ')'
|   PredicateExpr ( IsOrNotOp 'NULL' | CompareOp ( ( singleAtIdentifier assignmentEq )? PredicateExpr | AnyOrAll SubSelect ) )* ( IsOrNotOp ( trueKwd | falseKwd | 'UNKNOWN' ) )?

PredicateExpr ::=
    BitExpr ( BetweenOrNotOp BitExpr 'AND' BitExpr )* ( InOrNotOp ( '(' ExpressionList ')' | SubSelect ) | LikeOrNotOp SimpleExpr LikeEscapeOpt | RegexpOrNotOp SimpleExpr )?

BitExpr ::=
    BitExpr ( ( '|' | '&' | '<<' | '>>' | '*' | '/' | '%' | 'DIV' | 'MOD' | '^' ) BitExpr | ( '+' | '-' ) ( BitExpr | "INTERVAL" Expression TimeUnit ) )
|   SimpleExpr

SimpleExpr ::=
    SimpleIdent ( ( '->' | '->>' ) stringLit )?
|   FunctionCallKeyword
|   FunctionCallNonKeyword
|   FunctionCallGeneric
|   SimpleExpr ( 'COLLATE' CollationName | pipes SimpleExpr )
|   WindowFuncCall
|   Literal
|   paramMarker
|   Variable
|   SumExpr
|   ( '!' | '~' | '-' | '+' | 'NOT' | 'BINARY' ) SimpleExpr
|   'EXISTS'? SubSelect
|   ( ( '(' ( ExpressionList ',' )? | 'ROW' '(' ExpressionList ',' ) Expression | builtinCast '(' Expression 'AS' CastType | ( 'DEFAULT' | 'VALUES' ) '(' SimpleIdent | 'CONVERT' '(' Expression ( ',' CastType | 'USING' CharsetName ) ) ')'
|   'CASE' ExpressionOpt WhenClause+ ElseOpt 'END'
```