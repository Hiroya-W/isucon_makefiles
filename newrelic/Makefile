PROJECT_ROOT:=/home/isucon/webapp

.PHONY: marker
marker: deployment_marker.sh
	@echo "\e[32mデプロイメントマーカーを設定します\e[m"
	sh $(PROJECT_ROOT)/webapp/deployment_marker.sh
	@echo "\n"

deployment_marker.sh:
	@echo "\e[32mdeployment_marker.shを取得します\e[m"
	wget https://gist.githubusercontent.com/Hiroya-W/9597394bc0a01f67ec8a596d8780a397/raw/1936ef21baa55efdc76803c0c64502efb81bbd88/deployment_marker.sh -O deployment_marker.sh

