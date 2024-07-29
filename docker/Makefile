.PHONY: deps run clean


deps:
	# https://github.com/lizrice/containers-from-scratch/issues/10
	docker run -d --rm --name ubuntufs ubuntu:18.04 sleep 1000
	docker export ubuntufs -o ./src/ubuntufs.tar
	docker stop ubuntufs


run: deps
	cd vm && vagrant up && vagrant ssh
	

clean:
	rm -rf ./src/ubuntufs*
