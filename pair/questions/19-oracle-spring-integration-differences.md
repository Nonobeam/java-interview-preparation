## Q19: What is different about Oracle when integrating with Spring, compared to PostgreSQL/MySQL?

Not generic SQL differences — focus on Spring/JPA integration: SEQUENCE vs IDENTITY, VARCHAR2 4000 limit + CLOB, no boolean type, empty string = NULL, stored procedures with packages and IN/OUT params, DUAL table, FOR UPDATE WAIT/NOWAIT, DATE includes time.
