all: build

build:
	docker build -t araalinetworks/struts -f Dockerfile .

publish:
	docker tag araalinetworks/struts quay.io/araalinetworks/struts
	docker push quay.io/araalinetworks/struts
