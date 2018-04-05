docker login

mvn install -Denv.docker.image.prefix=yadneshchonkar

mvn dockerfile:push
mvn deploy

URLS
http://localhost:8091/spikeCPU?loopSize=10
http://localhost:8091/spikeMemory?words=10&length=10
http://localhost:8091/sleep?time=10


http://localhost:8091/health
http://localhost:8091/metrics
http://localhost:8091/info
http://localhost:8091/trace