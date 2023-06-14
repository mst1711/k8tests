VIDEOFOLDER?=./video
REDISPV?=./redis
LINK_TTL?=604800 # 7 nights

debug:
	docker run --name openresty-debug \
		--rm \
		-it \
		--entrypoint=/bin/bash \
		--network $(DOCKER_NETWORK) \
		$(IMAGE)

run: start filldb

start:
	@mkdir -p $(VIDEOFOLDER)
	@mkdir -p $(REDISPV)
	@docker-compose up -d

stop:
	@docker exec -it redis sh -c "redis-cli save"
	@docker-compose down

filldb:
	@rm -f $(VIDEOFOLDER)/*
	@docker exec -it redis sh -c "echo FLUSHDB SYNC | redis-cli" 2>&1 > /dev/null
	@wget --quiet https://jsoncompare.org/LearningContainer/SampleFiles/Video/MP4/Sample-MP4-Video-File-for-Testing.mp4 -O $(VIDEOFOLDER)/1.mp4
	@wget --quiet https://joy1.videvo.net/videvo_files/video/free/2018-08/originalContent/180626_12_Seocho-Dong_20.mp4 -O $(VIDEOFOLDER)/2.mp4
	@wget --quiet https://download.samplelib.com/mp4/sample-30s.mp4 -O $(VIDEOFOLDER)/3.mp4
	@wget --quiet https://joy1.videvo.net/videvo_files/video/free/2014-12/originalContent/Raindrops_Videvo.mp4 -O $(VIDEOFOLDER)/4.mp4
	@for file in $$(ls $(VIDEOFOLDER)); do \
		docker exec -it redis sh -c "redis-cli set /$$file yes EX $(LINK_TTL)" 2>&1 > /dev/null; \
		echo "Now you can use link: http://127.0.0.1/video/$$file"; \
	done
