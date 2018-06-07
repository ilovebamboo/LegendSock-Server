Legendsock 2.2 订阅功能添加步骤

Legendsock 2.2 订阅功能添加步骤
Legendsock 2.2 订阅功能添加步骤
2017.09.23 22:55 UPDATE

首先，建立文件，subscribe.php 在WHMCS 根目录，

内容：

<?php
require_once 'init.php';
require_once 'modules/addons/legendsock/class.php';function base64url_encode($data){return rtrim(strtr(base64_encode($data),'+/','-_'),'=');}functionGetRandStr( $len ){ 
    $chars =["a","b","c","d","e","f","g","h","i","j","k","l","m","n","o","p","q","r","s","t","u","v","w","x","y","z","A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","0","1","2","3","4","5","6","7","8","9"]; 
    $charsLen = count($chars)-1; 
    shuffle($chars);   
    $output ="";for($i=0; $i<$len; $i++){ 
        $output .= $chars[mt_rand(0, $charsLen)];}return $output;}if( $_REQUEST['action']=='reSet'){
	$newpasswd 	=GetRandStr(12);
	$sid 		=(int) $_REQUEST['sid'];
	$userid 	=(int) $_REQUEST['uid'];
	$data		= \WHMCS\Database\Capsule::table('tblhosting')->where('id', $sid)->first();if( $userid != $data->userid or empty($userid)){
		$result =['status'=>'error','msg'=>'参数错误',];}else{
		$result 	= \WHMCS\Database\Capsule::table('tblhosting')->where('id', $sid)->update(['dedicatedip'=> $newpasswd]);if( empty( $result )){
		    $result =['status'=>'error','msg'=>'重置失败',];}else{
			$result =['status'=>'success','msg'=> $newpasswd,];}}
	echo json_encode($result);die();}

$sid 	=(int) $_GET['sid'];
$token 	= $_GET['token'];

$product 	= \WHMCS\Database\Capsule::table('tblhosting')->where('dedicatedip', $token)->where('id', $sid)->first();if( empty( $product )){die('什么也没输出');}if( $token == $product->dedicatedip ){

	$hosts = \WHMCS\Database\Capsule::table('tblproducts')->where('id', $product->packageid)->first()->configoption12;
	$servers = \WHMCS\Database\Capsule::table('ls_setting')->where('sid', $product->server)->first()->node;if(!empty( $hosts )){
		$hosts = explode(PHP_EOL, $hosts);}else{if(!empty( $servers )){
			$hosts = explode(PHP_EOL, $servers);}}
	
	$i =0;foreach($hosts as $host){
		list(, $hosts[$i])= explode('|', $host);
		list($remark[$i])= explode('|', $host);++$i;}
	
	$i =0;
	$ls =new \LegendSock\Extended();
	$db =new \LegendSock\Database();
	$data = $ls->getConnect($product->server);
	$getData = $data->runSQL(['action'=>['user'=>['sql'=>'SELECT u,d,t,port,obfs,method,protocol,passwd,transfer_enable FROM user WHERE pid = ?','pre'=>[
					$product->id
				]]],'trans'=>false]);if(empty($getData['user']['result'])){thrownewException('无法从数据库中取得当前产品的信息，请检查产品是否并未处于开通状态');}
	
	$get = $getData['user']['result'];
	
	$output = array();foreach($hosts as $host){
		$temp['remark']= $remark[$i++];
		$temp['port']= $get['port'];
		$temp['hostname']= $host;
		$temp['password']= $get['passwd'];
		$temp['obfs']= $get['obfs'];
		$temp['method']= $get['method'];
		$temp['protocol']= $get['protocol'];
		array_push($output, $temp);}
	
	$text ='';foreach($output as $val){
		$code = $val['hostname'].':'. $val['port'].':'. $val['protocol'].':'. $val['method'].':'. $val['obfs'].':'. base64_encode($temp['password']);
		$result .='ssr://'. base64url_encode( $code .'/?obfsparam=&remarks='. base64url_encode($val['remark']).'&group='.base64url_encode($GLOBALS['CONFIG']['CompanyName']).'&udpport=0&uot=0'). PHP_EOL;}if( $product->domainstatus =='Active'){exit(base64_encode($result));}}
然后编辑LS 2.2 客户端模板 WHMCS/modules/server/legendsock/templates/主题名称/client.tpl

在页面内加入：

<linkrel="stylesheet"href="{$systemurl}modules/servers/legendsock/templates/NeWorld/sweetalert.css?v2"><scripttype="text/javascript"src="{$systemurl}modules/servers/legendsock/templates/NeWorld/sweetalert.min.js"></script>
{literal}
<script>var completeFlag =true;functionSubScribe(sID, token){if( token==undefined){
			subscribeurl ="{/literal}{$LS_LANG['subscribe']['ResetURL']}{literal}";}else{
			subscribeurl ="{/literal}{$systemurl}{literal}subscribe/"+sID+"/"+token+"/";}
		swal({
			title:"{/literal}{$LS_LANG['subscribe']['url']}{literal}",
			text: subscribeurl,
			type:"info",
			showCancelButton:true,
			closeOnConfirm:false,
			showLoaderOnConfirm:true,
			cancelButtonText:"{/literal}{$LANG.cancel}{literal}",
			confirmButtonText:"{/literal}{$LS_LANG['subscribe']['Reset']}{literal}",},function(){if(!completeFlag){return;}
			$.ajax({
				method:"GET",
				url:"{/literal}{$systemurl}{literal}subscribe.php?action=reSet",
				data:{sid: sID, uid:'{/literal}{$clientsdetails['userid']}{literal}'},
				dataType:'json',
				cache:false,
				beforeSend:function(){
					completeFlag =false;},
				complete:function(){
					completeFlag =true;},
				success:function(data){if(data.status=='success'){
						swal({
							title:"{/literal}{$LS_LANG['subscribe']['ResetSuccess']}{literal}",
							text:"{/literal}{$systemurl}{literal}subscribe/"+sID+"/"+data.msg+"/",
							type:"success"});}elseif(data.status=='error'){
						swal({
							title:"{/literal}{$LS_LANG['subscribe']['ResetError']}{literal}",
							text: data.msg,
							type:"error"});};},
				error:function(){
					swal("服务器忙，请稍后重试");}});});}</script>
{/literal}
适当的地方加入：

<divclass="col-md-4 col-sm-6"><divclass="box"><h3>{$LS_LANG['subscribe']['title']}</h3><ul><li><strong>{$LS_LANG['subscribe']['url']}</strong><divclass="btn-group btn-group-xs"role="group"aria-label="Extra-small button group"><aclass="btn btn-success btn-xs"onClick="javascript:SubScribe('{$serviceid}','{$dedicatedip}');"><spanclass="glyphicon glyphicon-bookmark"aria-hidden="true"></span> {$LS_LANG['subscribe']['url']}
		                            </a></div></li><li><strong>{$LS_LANG['plugin']['guiconfig']}</strong><buttontype="button"class="btn btn-primary btn-xs autohides"name="guiconfig"data-guiconfig="{$guiconfig['ssr']}"><spanclass="glyphicon glyphicon-export"aria-hidden="true"></span> {$LS_LANG['plugin']['ssr']}
	                            </button></li><li><strong>Surge</strong><buttontype="button"class="btn btn-primary btn-xs autohides"name="guiconfig"data-guiconfig="{$guiconfig['ssr']}"><spanclass="glyphicon glyphicon-export"aria-hidden="true"></span> {$LS_LANG['plugin']['ssr']}
	                            </button></li></ul></div></div>
切记 将 sweetalert 的 css 和 js 放入 使用的模板目录下！请自行下载 SweetAlert 上传！

WHMCS/modules/servers/legendsock/languages/zh_CN.php

新增语言包：

'subscribe'=>['title'=>'订阅信息','url'=>'节点订阅','ResetURL'=>'请点击重置您的订阅地址','Reset'=>'重置','ResetSuccess'=>'订阅地址重置成功','ResetError'=>'订阅地址重置失败',],
记得最后增加伪静态规则

Apache 伪静态规则
RewriteRule^subscribe/([^/]*)?/?([^/]*)?/?$ ./subscribe.php?sid=$1&token=$2 [QSA,L]
Nginx 伪静态规则
rewrite ^/subscribe/([^/]*)?/?([^/]*)?/?$ /./subscribe.php?sid=$1&token=$2 last;
