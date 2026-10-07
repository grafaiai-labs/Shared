[dev@ip-172-16-25-94 sandbox]$ sed -i 's/^FRONTEND_BIND_ADDRESS=.*/FRONTEND_BIND_ADDRESS=0.0.0.0/' .env
[dev@ip-172-16-25-94 sandbox]$ docker compose up -d
[+] up 2/2
 ✔ Container rag-poc-backend-1  Healthy                                                                                                                           18.0s
 ✔ Container rag-poc-frontend-1 Started                                                                                                                           17.3s
[dev@ip-172-16-25-94 sandbox]$ docker ps --format '{{.Names}}\t{{.Ports}}'
rag-poc-frontend-1      0.0.0.0:8081->8080/tcp
rag-poc-backend-1       8000/tcp
superset_chat_agent     8000/tcp
superset_app    0.0.0.0:8080->8088/tcp
superset_cube   4000/tcp
superset_db     5432/tcp
superset_cache  6379/tcp
superset_worker 8088/tcp
superset_worker_beat    8088/tcp
