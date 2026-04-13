## Q3: @Transactional propagation levels deep dive

Explain all 7 propagation levels (REQUIRED, REQUIRES_NEW, NESTED, SUPPORTS, NOT_SUPPORTED, MANDATORY, NEVER) with practical scenarios. Cover: REQUIRED's rollback-only trap when catching inner exceptions, REQUIRES_NEW's deadlock risk, NESTED vs REQUIRES_NEW differences (savepoint vs separate connection), and when to use each level.
