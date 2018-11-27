.PHONY: all converge install verify test

BE=bundle exec
PR=pipenv run

all: test

install: bundle_install pipenv_install ansible-galaxy_install

lint: ansible-lint yamllint rubocop

pre_converge: ansible-lint yamllint

pre_verify: rubocop

ansible-lint:
	$(PR) ansible-lint .

yamllint:
	$(PR) yamllint .

rubocop:
	$(BE) rubocop ./test/**/*.rb

create:
	$(BE) kitchen create

converge:
	$(BE) kitchen converge

destroy:
	$(BE) kitchen destroy

login:
	$(BE) kitchen login

test:
	$(BE) kitchen test

verify:
	$(BE) kitchen verify

bundle_install:
	bundle install

pipenv_install:
	pipenv install

ansible-galaxy_install:
	if [ -f ./requirements.yml ] ; \
	then \
		ansible-galaxy install --force -r requirements.yml ; \
	fi
