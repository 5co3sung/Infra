# 03_ha_failover

## 포함 문서
- [keepalived_vip_failover.md](./keepalived_vip_failover.md)
- [postgresql_replication_and_pg_auto_failover.md](./postgresql_replication_and_pg_auto_failover.md)
- [mariadb_mha_overview.md](./mariadb_mha_overview.md)
- [redis_sentinel_operations.md](./redis_sentinel_operations.md)

## 이 카테고리의 의미
Keepalived는 Nginx 하위 기능이 아니라 별도의 고가용성(HA) 구성 요소입니다.
DB 이중화, Failover, Sentinel도 모두 같은 성격으로 묶는 편이 구조상 자연스럽습니다.
