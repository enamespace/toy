.PHONY: deps run

deps:
	sudo apt install make
	sudo snap install go --classic
	mkdir -p /vagrant_data/ubuntufs
	cd /vagrant_data && tar xf ubuntufs.tar -C ./ubuntufs
	
run: 
	# ps fax
	# cat /sys/fs/cgroup/pids/ubuntufs-group/pids.max 
	sudo go run main.go run /bin/bash 
	
