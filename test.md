cd ~/work/poc-feature-rag/deploy/sandbox
grep -E '^FRONTEND_(HOST_PORT|BIND_ADDRESS|ALLOWED_PORT_RANGE)=' .env    # ①
docker compose ps --format '{{.Service}}\t{{.Status}}'                   # ②
docker ps --format '{{.Names}}\t{{.Ports}}'                              # ③
curl -s -o /dev/null -w 'local:   %{http_code}\n' http://127.0.0.1:8081/api/health                           # ④
curl -s -o /dev/null -w 'private: %{http_code}\n' "http://$(hostname -I | awk '{print $1}'):8081/api/health"  # ⑤
hostname -I | awk '{print $1}'                                           # ⑥ 結果は貼らず、Windows で開いている IP と同じかだけ教えてください
docker compose logs --tail=20 frontend                                   # ⑦


Test-NetConnection <Linux の Private IP> -Port 8081 | Select-Object TcpTestSucceeded
Test-NetConnection <Linux の Private IP> -Port 8080 | Select-Object TcpTestSucceeded
