DATE:=$(shell date +%Y%m%d-%H%M%S)
CD:=$(CURDIR)
PROJECT_ROOT:=/home/isucon/webapp

# env.shに合わせて変更する
DB_HOST:=127.0.0.1
DB_PORT:=3306
DB_USER:=isucon
DB_PASS:=isucon
DB_NAME:=isucholar

MYSQL_CMD:=mysql -h$(DB_HOST) -P$(DB_PORT) -u$(DB_USER) -p$(DB_PASS) $(DB_NAME)

NGINX_LOG:=/var/log/nginx/access.log
MYSQL_LOG:=/var/log/mysql/slow-query.log

.PHONY: bench-%
bench-%: pull-%
	make build
	make etc
	make log-reset
	make restart slow-on
	@echo "\e[32mベンチの準備が完了しました\e[m"

.PHONY: bench
bench: build etc log-reset restart slow-on
	@echo "\e[32mベンチの準備が完了しました\e[m"

.PHONY: pull-%
pull-%:
	@echo "\e[32mgit pull\e[m"
	git fetch origin
	git checkout origin/$(patsubst pull-%,%,$@)
	git rebase main

.PHONY: build
build:
	@echo "\e[32mNode.js appをbuildします\e[m"
	cd nodejs && npm run build

.PHONY: etc
etc:
	@echo "\e[32m/etc にファイルを配置します\e[m"
	sudo rm -rf /etc/mysql
	sudo rm -rf /etc/nginx
	sudo cp -r mysql /etc/mysql
	sudo cp -r nginx /etc/nginx

.PHONY: etc-backup
etc-backup:
	@echo "\e[32m/etc を取得します\e[m"
	sudo mv /etc/mysql mysql
	sudo mv /etc/nginx nginx

.PHONY: cat
cat: cat-msg slow-log alp

.PHONY: cat-msg
cat-msg:
	git log -1 --pretty=format:"%s %h" | discocat

.PHONY: slow-log
slow-log:
	@echo "\e[32mslow-logsを出力します\e[m"
	sudo pt-query-digest --limit=100 --filter '$$event->{bytes} <= 100000' $(MYSQL_LOG) | discocat

.PHONY: mysqlslowdump
mysqlslowdump:
	sudo mysqldumpslow -s t $(SLOW_LOG) | head -n 20 | discocat

.PHONY: log-reset
log-reset:
	@echo "\e[32mlogファイルを初期化します\e[m"
	mkdir -p ~/logs/$(DATE)
	-sudo mv -f $(MYSQL_LOG) ~/logs/$(DATE)/slow-query.txt
	-sudo mv -f $(NGINX_LOG) ~/logs/$(DATE)/access.txt

ALPSORT=sum
ALPM="^/api/courses/[A-Z0-9]+,^/api/courses/.+/classes,^/api/announcements/.+,^/api/announcements\?page"
.PHONY: alp
alp:
	@echo "\e[32maccess logをalpで出力します\e[m"
	sudo cat $(NGINX_LOG) | alp ltsv --sort $(ALPSORT) -m $(ALPM) --reverse -q | discocat

.PHONY: restart
restart:
	@echo "\e[32mサービスを再起動します\e[m"
	sudo systemctl restart mysql.service
	sudo systemctl restart nginx.service
	sudo systemctl restart isucholar.nodejs.service

.PHONY: slow-on
slow-on:
	@echo "\e[32mMySQL slow-querry ONにします\e[m"
	sudo mysql -e "set global slow_query_log_file = '$(MYSQL_LOG)'; set global long_query_time = 0; set global slow_query_log = ON;"
	sudo mysql -e "show variables like 'slow%';"

.PHONY: slow-off
slow-off:
	@echo "\e[32mMySQL slow-querry OFFにします\e[m"
	sudo mysql -e "set global slow_query_log = OFF;"
	sudo mysql -e "show variables like 'slow%';"
