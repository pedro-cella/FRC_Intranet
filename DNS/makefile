dkc := "docker-compose.yml"

start: 
	docker-compose -f ${dkc} up --build
	docker exec  machine1 ./config.sh
	