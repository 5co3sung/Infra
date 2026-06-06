# 03_ha_failover

서비스 연속성을 확보하기 위한 고가용성(HA) 및 장애 전환(Failover) 구성 요소를 정리한 영역입니다.
웹 계층의 VIP 전환부터 DB/Redis 이중화까지 운영자가 점검해야 하는 흐름을 다룹니다.

## Documents

- [keepalived_vip_failover.md](./keepalived_vip_failover.md)
- [postgresql_replication_and_pg_auto_failover.md](./postgresql_replication_and_pg_auto_failover.md)
- [mariadb_mha_overview.md](./mariadb_mha_overview.md)
- [redis_sentinel_operations.md](./redis_sentinel_operations.md)

## Key Points

- Keepalived 기반 VIP Failover 구성 이해
- PostgreSQL 복제와 pg_auto_failover 검토
- MariaDB MHA, Redis Sentinel 운영 체크포인트 정리

