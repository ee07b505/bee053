#!/bin/bash
Workspace=/root/aleo
ScreenName=aleo
KeyFile="/root/my_aleo_key.txt"
has_private_key(){
	PrivateKey=$(cat ${KeyFile} | grep "Private Key" | awk '{print $3}')	
	if [ -z "${PrivateKey}" ]
	then
		echo "密钥不存在！"
		return 1
	else
		echo "密钥可正常读取"
		return 0
	fi
}
generate_key(){
	echo "开始生成账户密钥"
	snarkos account new > ${KeyFile}

	has_private_key || exit 1

	# 先将可能存在于/etc/profile中的密钥记录删除
	sed -i '/PROVER_PRIVATE_KEY/d' /etc/profile

	# 将密钥保存/etc/profile中使得可以被启动脚本读取
	PrivateKey=$(cat ${KeyFile} | grep "Private Key" | awk '{print $3}')
	echo "export PROVER_PRIVATE_KEY=$PrivateKey" >> /etc/profile
	source /etc/profile
}

has_screen(){
	Name=`screen -ls | grep ${ScreenName}`
	if [ -z "${Name}" ]
	then
		return 1
	else
		echo "screen 运行中：${Name}"
		return 0
	fi
}

run_prover(){
	source $HOME/.cargo/env
	source /etc/profile
	has_private_key || generate_key
	echo “账户和密钥保存在 ${KeyFile}，请妥善保管，以下是详细信息：”
	cat ${KeyFile}
	cd ${Workspace}

	# 判断是否已经有screen存在了
        has_screen && echo "执行screen -D -r aleo 进入screen查看" && exit 1
	# 判断是否有密钥
        has_private_key || exit 1

	# 启动一个screen,并在screen中启动prover节点
        screen -dmS ${ScreenName}
        cmd=$"./run-prover.sh"
        screen -x -S ${ScreenName} -p 0 -X stuff "${cmd}"
        screen -x -S ${ScreenName} -p 0 -X stuff $'\n'
        echo "client节点已在screen中启动，可执行screen -D -r aleo 来查看节点运行情况"
}
run_prover
