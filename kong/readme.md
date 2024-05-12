#Build & run
run kong-gateway
docker run -d --name kong-database \
 --network=kong-net \
 -p 5432:5432 \
 -e "POSTGRES_USER=kong" \
 -e "POSTGRES_DB=kong" \
 -e "POSTGRES_PASSWORD=kongpass" \
 postgres:9.6

docker run --rm --network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_PASSWORD=kongpass" \
-e "KONG_PASSWORD=test" \
kong/kong-gateway:2.6.1.0-alpine kong migrations bootstrap


docker run -d --name kong-gateway \
--network=kong-net \
-v "/home/tuongncn/repository-git/git/tcbs-rate-limiting:/usr/local/custom" \
-e "KONG_LUA_PACKAGE_PATH=/usr/local/custom/?.lua;;" \
-e "KONG_PLUGINS=bundled,tcbs-rate-limiting" \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_USER=kong" \
-e "KONG_PG_PASSWORD=kongpass" \
-e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
-e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
-e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
-e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
-p 8000:8000 \
-p 8443:8443 \
-p 8001:8001 \
-p 8444:8444 \
-p 8002:8002 \
-p 8445:8445 \
-p 8003:8003 \
-p 8004:8004 \
kong/kong-gateway:2.3.3.0-alpine


#Test

http :8000/api "X-Forwarded-For: 192.168.200.129, 70.41.3.18, 150.172.238.178"

http DELETE localhost:8001/services/demo_plugin/plugins name=tcbs-rate-limiting

curl -X POST http://localhost:8001/services/demo_service/plugins \
   --data "name=tcbs-rate-limiting" \
   --data config.minute=5 \
   --data config.policy=local \
   --data config.limit_by=ip \
   --data config.whitelist_ips=192.168.0.0/24

curl -X POST http://localhost:8001/services/demo_service/plugins \
   --data "name=tcbs-rate-limiting" \
   --data config.minute=5 \
   --data config.policy=local \
   --data config.limit_by=ip \
   --data config.whitelist_ips=192.168.200.129


curl -X POST http://localhost:8001/services/demo_service/plugins \
   --data "name=tcbs-rate-limiting" \
   --data config.minute=5 \
   --data config.policy=local \
   --data config.limit_by=header \
   --data config.header_name=xxxx \
   --data "config.whitelist_ips=192.168.200.129"

