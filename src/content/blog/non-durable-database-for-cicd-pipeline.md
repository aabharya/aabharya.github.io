---
author: Ali Abharya
pubDatetime: 2025-03-06T17:47:37.939Z
modDatetime: 2025-03-06T17:47:37.939Z
title: Non Durable Database for CI/CD Pipeline
slug: non-durable-database-for-cicd-pipeline
featured: true
draft: false
tags:
  - postgresql
  - django
  - database
  - ci/cd
  - unit test

description: How to apply non durable config to postgres in CI/CD pipeline
---

# The idea of non-durable database for CI/CD pipeline

Recently, I started studying **Designing Data-Intensive Applications (DDAI)**, and one of its core concepts is durability. This made me ask myself:

`When and where do we not need durability?`

To explore this, I experimented with non-durable database settings while running unit tests, hoping to speed up the CI/CD pipeline.

![ddai](@assets/images/ddai_sm.png)

## Introduction

**Continuous Integration and Continuous Deployment** **(CI/CD)** pipelines rely on **fast and efficient** test execution.

Databases like PostgreSQL provide durability guarantees to ensure data safety, but in a test environment, these guarantees are often unnecessary.

By disabling durability settings and using unlogged tables, we could significantly speed up test execution.

In this article, I'll cover:

1. What is durability in databases and why it matters?
2. Non durable settings for PostgreSQL?
3. What are unlogged tables in PostgreSQL?
4. How to implement unlogged tables in Django ORM?
5. Benchmarking unit test execution with non durable database settings

---

## 1️⃣ What Is Durability in Databases?

The **ACID** principles of databases ensure data integrity:

- **Atomicity**: Transactions are all-or-nothing.
- **Consistency**: Transactions take the database from one valid state to another.
- **Isolation**: Transactions do not interfere with each other.
- **Durability**: Once a transaction is committed, it must persist, even after a crash.

### **Why Durability Matters?**

In production environments, durability ensures that data is not lost even if the system crashes.
PostgreSQL achieves this using **Write-Ahead Logging (WAL)**, **fsync**, and other mechanisms.

However, in **CI/CD pipelines**, we care about **fast test execution** and data loss is **irrelevant** since the database is destroyed after tests.

---

## 2️⃣ Non durable settings for PostgreSQL

We can modify `postgresql.conf` to disable durability. Here’s what each setting does:

```ini
    # Disable WAL and disk synchronization to speed up tests (non-durable settings)
    fsync = off
    synchronous_commit = off
    full_page_writes = off

    # Disable background writer to reduce I/O
    bgwriter_lru_maxpages = 0

    # Reduce checkpoints to avoid unnecessary writes
    checkpoint_timeout = 1h
    max_wal_size = 2GB
    min_wal_size = 512MB

    # Reduce logging overhead
    logging_collector = off
    log_min_duration_statement = -1
    log_statement = 'none'

    # Disable autovacuum to avoid performance impact during tests
    autovacuum = off

    # Allow more connections for parallel test runs
    max_connections = 100

    # Increase shared buffers for better performance
    shared_buffers = 128MB

    # Reduce work memory to avoid excessive RAM usage in tests
    work_mem = 4MB
```

Disabling these settings makes writes nearly instant because PostgreSQL no longer waits for disk I/O.
This could cut down test execution time in CI/CD pipelines.

Using a docker compose we could apply these settings:

```yaml
# file: docker-compose-ci.yaml
services:
  db:
    image: "postgres:13-alpine"
    restart: "no"
    environment:
      POSTGRES_USER: ...
      POSTGRES_PASSWORD: ...
      PGPASSWORD: ...
      POSTGRES_DB: ...
      PGDATA: /db_data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - BACKEND
    tmpfs:
      - /db_data
    volumes:
      - ./deploy/db/test_db.conf:/etc/postgresql/postgresql.conf
```

---

## 3️⃣ What Are Unlogged Tables in PostgreSQL

Unlogged tables are a PostgreSQL feature that further improves performance by disabling WAL (Write-Ahead Logging) entirely.

### Benefits of Unlogged Tables

- No WAL logging → Faster inserts, updates, and deletes.
- Ideal for temporary/test data → No need to persist data.
- Survive crashes, but not restarts → Tables are emptied if the server restarts.

### Downsides

- No crash recovery → Data is lost if PostgreSQL restarts.
- No replication → Cannot be replicated to another server.
- No foreign key constraints to logged tables → Must be used carefully.

### How to Create an Unlogged Table?

```sql
    CREATE UNLOGGED TABLE test_results (
    id SERIAL PRIMARY KEY,
    result TEXT NOT NULL
);

```

---

## 4️⃣ Implementing Unlogged Tables with Django ORM

**Django ORM** does not support unlogged tables out of the box, but we can achieve this by overriding the database schema editor.

- Create a new python package and call it `test_db_engine`:
- Create a new python module inside it and call it `base.py`
- Create a custom database schema editor class and apply it to a custom database wrapper:

```python
    # file: test_db_engine/base.py
    from django.db.backends.postgresql.base import DatabaseWrapper as PostgresDatabaseWrapper
    from django.db.backends.postgresql.schema import DatabaseSchemaEditor


    class CreateUnloggedTableSchemaEditor(DatabaseSchemaEditor):
        sql_create_table = 'CREATE UNLOGGED TABLE %(table)s (%(definition)s)'


    class DatabaseWrapper(PostgresDatabaseWrapper):
        SchemaEditorClass = CreateUnloggedTableSchemaEditor

```

- In your tests settings module use this engine for postgres:

```python
    # test_settings.py
    DATABASES['default']['engine'] = 'test_db_engine'
```

Applying these settings froces django to create unlogged tables when applying migrations during initial unit test setup.

---

## 5️⃣ Benchmarking unit test execution

When I first applied these changes locally, I ran the tests and got around **10 seconds improvement** in unit test execution. Keep in mind that I did not run my tests in local using docker compose, just regular command line execution as a separate process.

Excited by this boost, I pushed the changes to my remote repository to benchmark them in a real CI/CD pipeline.

To my surprise, I didn’t see any speed improvement in the pipeline. After some investigation, I discovered what was actually happening.

The key was hidden in the Docker Compose configuration, specifically this line:

```yaml
tmpfs:
  - /db_data
```

Using `tmpfs` for `/db_data` means the database is stored entirely in memory instead of being written to disk.

By default, PostgreSQL caches data in shared buffers and uses the OS page cache. However, with `tmpfs`, all data is forced to stay in memory.

This means my so-called **non-durable settings** were doing nothing because the database was already operating with maximum speed in memory.

```ini
    fsync = off
    full_page_writes = off
    bgwriter_lru_maxpages = 0
    autovacuum = off
```

## Key Takeaways

1. My tests weren’t stressing the database enough to expose the performance bottlenecks. I was trying to optimize the configurations that are related to disk pages while CI/CD pipeline database already runs in memory.
2. The real bottleneck was Django’s ORM overhead (e.g., object creation, serialization). Tweaking the database settings wasn’t the key issue, the Python code itself was the bigger factor.

## Lessons Learned

If you're looking for practical ways to speed up Django unit tests, I highly recommend checking out this book:

![speed_up_django_tests](@assets/images/speed_up_django_tests.png)

Still, my experiment was valuable. I learned a lot about database configuration. As **Uncle Johnny** said Once:

`There is no shame in learning, and you only learn by failures` 🙏🏻
