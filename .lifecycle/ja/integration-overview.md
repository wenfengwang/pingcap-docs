---
title: データ統合の概要
summary: データ統合シナリオの概要について学びます。
---

# データ統合の概要

データ統合とは、様々なデータソース間でのデータの流れ、転送、および統合のことを指します。データ量が指数関数的に増加し、データの価値がより深く探求されるにつれて、データ統合はますます人気が高まり、緊急性を増しています。TiDBがデータサイロになる状況を避け、異なるプラットフォームとのデータを統合するために、TiCDCはTiDBのインクリメンタルなデータ変更ログを他のデータプラットフォームにレプリケートする能力を提供します。この文書では、TiCDCを使用したデータ統合アプリケーションについて説明します。ビジネスシナリオに合った統合ソリューションを選択してください。

## Confluent CloudおよびSnowflakeとの統合

TiCDCを使用して、TiDBからのインクリメンタルデータをConfluent Cloudにレプリケートし、Confluent Cloudを介してSnowflake、ksqlDB、SQL Serverにデータをレプリケートすることができます。詳細は、[Confluent CloudおよびSnowflakeとの統合](/ticdc/integrate-confluent-using-ticdc.md)を参照してください。

## Apache KafkaおよびApache Flinkとの統合

TiCDCを使用して、TiDBからのインクリメンタルデータをApache Kafkaにレプリケートし、Apache Flinkを使用してデータを消費することができます。詳細は、[Apache KafkaおよびApache Flinkとの統合](/replicate-data-to-kafka.md)を参照してください。