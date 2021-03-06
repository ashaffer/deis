include ../includes.mk

# the filepath to this repository, relative to $GOPATH/src
repo_path = github.com/deis/deis/logspout

GO_FILES = attacher.go logspout.go routes.go types.go
GO_PACKAGES = utils
GO_PACKAGES_REPO_PATH = $(addprefix $(repo_path)/,$(GO_PACKAGES))

COMPONENT = $(notdir $(repo_path))
DOCKER_IMAGE := deis/$(COMPONENT)
RELEASE_IMAGE := $(DOCKER_IMAGE):$(BUILD_TAG)
DEV_DOCKER_IMAGE := $(DEV_REGISTRY)/$(RELEASE_IMAGE)

build: check-docker
	GOOS=linux GOARCH=amd64 CGO_ENABLED=0 godep go build -a -v -ldflags '-s' -o image/logspout
	docker build -t $(RELEASE_IMAGE) image

clean:
	rm -rf image/logspout

full-clean:
	docker images -q $(DOCKER_IMAGE) | xargs docker rmi -f
	docker images -q $(DEV_DOCKER_IMAGE) | xargs docker rmi -f
	docker images -q $(RELEASE_IMAGE) | xargs docker rmi -f

install: check-deisctl
	deisctl install $(COMPONENT)

uninstall: check-deisctl
	deisctl uninstall $(COMPONENT)

start: check-deisctl
	deisctl start $(COMPONENT)

stop: check-deisctl
	deisctl stop $(COMPONENT)

restart: stop start

run: install start

dev-release: push set-image

push: check-registry
	docker tag -f $(RELEASE_IMAGE) $(DEV_DOCKER_IMAGE)
	docker push $(DEV_DOCKER_IMAGE)

set-image: check-deisctl
	deisctl config $(COMPONENT) set image=$(DEV_DOCKER_IMAGE)

release:
	docker push $(RELEASE_IMAGE)

deploy: build dev-release restart

setup-root-gotools:
	sudo GOPATH=/tmp/tmpGOPATH go get -u -v code.google.com/p/go.tools/cmd/cover
	sudo GOPATH=/tmp/tmpGOPATH go get -u -v code.google.com/p/go.tools/cmd/vet
	sudo rm -rf /tmp/tmpGOPATH

setup-gotools:
	go get -v github.com/golang/lint/golint

test: test-style test-unit test-functional

test-functional:
	@echo no functional tests

test-style:
# display output, then check
	$(GOFMT) $(GO_PACKAGES) $(GO_FILES)
	@$(GOFMT) $(GO_PACKAGES) $(GO_FILES) | read; if [ $$? == 0 ]; then echo "gofmt check failed."; exit 1; fi
	$(GOVET) $(repo_path) $(GO_PACKAGES_REPO_PATH)
	$(GOLINT) ./...

test-unit:
	@echo no unit tests
