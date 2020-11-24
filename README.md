![Citus Banner](/github-banner.png)

[![Slack Status](http://slack.citusdata.com/badge.svg)](https://slack.citusdata.com)
[![Latest Docs](https://img.shields.io/badge/docs-latest-brightgreen.svg)](https://docs.citusdata.com/)
[![Code Coverage](https://codecov.io/gh/citusdata/citus/branch/master/graph/badge.svg)](https://codecov.io/gh/citusdata/citus/branch/master/graph/badge.svg)

## Citus: Postgres at any scale

Citus is an open source Postgres extension for achieving high performance at any scale. If your Postgres database keeps growing and you worry that a single server might not be enough, then Citus is here to save the day.

By using Citus, you extend your PostgreSQL database with several new superpowers:

- **Distributed tables** are sharded across a cluster of PostgreSQL servers to combine their CPU, memory, storage and I/O capacity.
- **References tables** are replicated to all servers for joins and foreign keys from distributed tables and maximum read performance.
- **Parallel, distributed query engine** parallelizes SELECT, DML, and other operations on distributed tables across the cluster.
- **Columnar storage** compresses data, speeds up scans, and supports fast projections, both on regular and distributed tables.

Because Citus is an extension, you can always use it with the latest Postgres version. That means that you can avoid a costly migration and you can keep using the Postgres tools and extensions that you are already familiar with.

### Example Citus use cases

Citus is uniquely capable of scaling both analytical and transactional workloads with up to petabytes of data. There are several use cases in which Citus is the state-of-the-art solution and most commonly used:

- **[Customer-facing analytics dashboards](http://docs.citusdata.com/en/latest/use_cases/realtime_analytics.html)**:
  Citus enables you to build analytics dashboard that simultaneously ingest and process large amounts of data in the database and give subsecond response times even with a large number of concurrent users.
  
  The advanced parallel, distributed query engine in Citus combined with PostgreSQL features such as [array types](https://www.postgresql.org/docs/current/arrays.html), [JSONB](https://www.postgresql.org/docs/current/datatype-json.html), [lateral joins](https://heap.io/blog/engineering/postgresqls-powerful-new-join-type-lateral), and extensions like [HyperLogLog](https://github.com/citusdata/postgresql-hll) and [TopN](https://github.com/citusdata/postgresql-topn) allow you to build responsive analytics dashboards no matter how many customers or how much data you have.

  Example users: [Algolia](https://www.citusdata.com/customers/algolia), [Heap](https://www.citusdata.com/customers/heap)
  
- **[Time series data](http://docs.citusdata.com/en/latest/use_cases/timeseries.html)**:
  Citus enables you to process and analyze very large amounts of time series data. The biggest Citus clusters store well over a petabyte of time series data and ingest terabytes per day.
  
  Citus integrates seamlessly with [Postgres table partitioning](https://www.postgresql.org/docs/current/ddl-partitioning.html) and [pg_partman](https://www.citusdata.com/blog/2018/01/24/citus-and-pg-partman-creating-a-scalable-time-series-database-on-PostgreSQL/), which can speed up queries and writes on time series tables. You can take advantage of the parallel, distributed query engine for fast analytical queries, and use the built-in *columnar storage* to compress old partitions.
  
  Example users: [MixRank](https://www.citusdata.com/customers/mixrank), [Windows team](https://techcommunity.microsoft.com/t5/azure-database-for-postgresql/architecting-petabyte-scale-analytics-by-scaling-out-postgres-on/ba-p/969685)

- **[Software-as-a-service (SaaS) applications](http://docs.citusdata.com/en/latest/use_cases/multi_tenant.html)**:
  SaaS and other multi-tenant applications need to be able to scale their database as the number of tenants/customers grows. Citus enables you to transparently shard a complex, data model by the tenant dimension such to enable your database to grow along with your business.
  
  By distributing tables along a tenant ID column and co-locating data for the same tenant, Citus can horizontally scale complex (tenant-scoped) queries, transactions, and foreign key graphs. Reference tables and distributed DDL commands make database management a breeze compared to manual sharding. On top of that, you have a built-in distributed query engine for doing cross-tenant analytics inside the database.
  
  Example users: [Copper](https://www.citusdata.com/customers/copper), [Salesloft](https://fivetran.com/case-studies/replicating-sharded-databases-a-case-study-of-salesloft-citus-data-and-fivetran),

### Getting started with Citus

The fastest way to get started with Citus is to use the [Hyperscale (Citus)](https://docs.microsoft.com/en-us/azure/postgresql/quickstart-create-hyperscale-portal) deployment option in Azure Database for PostgreSQL or [set up Citus locally](http://docs.citusdata.com/en/latest/installation/single_machine.html).

#### Hyperscale (Citus) on Azure Database for PostgreSQL

You can get a fully-managed Citus cluster in minutes through the Hyperscale (Citus) deployment option in the [Azure Database for PostgreSQL](https://azure.microsoft.com/en-us/services/postgresql/) portal. Azure will manage your backups, high availability through auto-failover, software updates, monitoring, and more for all of your servers. To get started with Hyperscale (Citus), use the [Hyperscale (Citus) Quickstart](https://docs.microsoft.com/en-us/azure/postgresql/quickstart-create-hyperscale-portal) in the Azure docs.

### Usage example: Distributed tables

Once you have a cluster, all you need to do is convert your tables by calling the `create_distributed_table` UDF:

```sql
CREATE TABLE events (user_id int, event_id bigserial, event_time timestamptz default now(), data jsonb not null); 

-- each node will contain a different subset of users
SELECT create_distributed_table('events', 'user_id');
```

After this operation, queries for a specific user ID will be efficiently routed to a single worker node, while queries across user IDs will be parallelized across the cluster.

### Usage example: Columnar storage

To use columnar storage, all you need to do is add `USING columnar` to your `CREATE TABLE` statements and your data will be automatically compressed using the columnar access method.

```sql
CREATE TABLE events (event_id bigserial, event_time timestamptz default now(), data jsonb not null) USING columnar; 
```

When using columnar storage, you should only load data in batch using `COPY` or `INSERT..SELECT` to achieve good  compression. Update, delete and indexes are currently unsupported on columnar tables.

You can use columnar storage by itself, or in a distributed table to combine the benefits of compression and distributed query parallelism.

### Talk to Contributors and Learn More

<table class="tg">
<col width="45%">
<col width="65%">
<tr>
  <td>Documentation</td>
  <td>Try the <a
  href="https://docs.citusdata.com/en/stable/tutorials/multi-tenant-tutorial.html">Citus
  tutorial</a> for a hands-on introduction or <br/>the <a
  href="https://docs.citusdata.com">documentation</a> for
  a more comprehensive reference.</td>
</tr>
<tr>
  <td>Slack</td>
  <td>Chat with us in our community <a
  href="https://slack.citusdata.com">Slack channel</a>.</td>
</tr>
<tr>
  <td>Github Issues</td>
  <td>We track specific bug reports and feature requests on our <a
  href="https://github.com/citusdata/citus/issues">project
  issues</a>.</td>
</tr>
<tr>
  <td>Twitter</td>
  <td>Follow <a href="https://twitter.com/citusdata">@citusdata</a>
  for general updates and PostgreSQL scaling tips.</td>
</tr>
<tr>
  <td>Citus Blog</td>
  <td>Read our <a href="https://www.citusdata.com/blog/">Citus Data Blog</a>
  for posts on Postgres, Citus, and scaling your database.</td>
</tr>
</table>

### Contributing

Citus is built on and of open source, and we welcome your contributions. The [CONTRIBUTING.md](CONTRIBUTING.md) file explains how to get started developing the Citus extension itself and our code quality guidelines.
