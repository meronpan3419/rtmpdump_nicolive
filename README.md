# rtmpdump_nicolive
rtmpdump for http://live.nicovideo.jp/

## なにこれ
rtmpdump（librtmp）にニコニコ生放送用のnlplaynoticeを実装したパッチを当てたやつです

## 入れ方

作業環境：ubuntu14.10

$ git clone https://github.com/meronpan3419/rtmpdump_nicolive.git
$ cd rtmpdump_nicolive
$ make 
$ make install
Makefileのprefix弄ってないとlibrtmpを/libにcpするので適当にするかsudo make installで


/libにlibrtmp入れた時にパス通ってないことあるので、通しておく。

$ export LD_LIBRARY_PATH=/lib

ubuntuの場合アレなので、
/etc/X11/Xsession.optionsのuse-ssh-agentをno-use-ssh-agentにして
ログインし直すとexportできるかも。

## how to use
http://watch.live.nicovideo.jp/api/getplayerstatus?v=lv******** から

$ rtmpdump -o out.flv -vr "[getplayerstatus/rtmp/url]/[getplayerstatus/stream/id]" -C S:"[getplayerstatus/rtmp/ticket]" -N "[getplayerstatus/stream/contents_list/contentsのrtmp://から始まる文字列]"

$ rtmpdump -o out.flv -vr "rtmp://nleu12.live.nicovideo.jp:1935/liveedge/live_150425_22_1/lv218887875" -C S:"155149:lv218887875:4:1429968658:686df8fd057d0825" -N "rtmp://nlpoca104.live.nicovideo.jp:1935/publicorigin/150425_22_1/,lv218887875?1429968658:30:5c5ce9d9d515fb0a" -p "http://live.nicovideo.jp/watch/lv218887875" 

###ファイルに保存する場合
$ rtmpdump ほにゃにゃら -o out.flv
###vlcで見る場合
$ rtmpdump ほにゃらら -o - | vlc -

## おまけ

$php watch.phpでマイページから自分の放送番組lvを取得してvlcで視聴



	<?php
	
	
		$id = "userid@exsample.com";
		$password = "pass";

		$user_session = file_get_contents("user_session");
	
		$contents = GET("http://live.nicovideo.jp/my", $user_session);
		if(!strstr($contents, '<title>マイページ')){
			Login($id, $password);
		}	

		$URI_MYPAGE = "http://live.nicovideo.jp/my";
	
		$contents = GET($URI_MYPAGE, $user_session);

		$lv = "";	

		if(preg_match("#http://live.nicovideo.jp/watch/(lv[0-9]+)(.*?)title=\"生放送ページへ戻る\"#", $contents, $match)){
			$lv =  $match[1];  
		}

		echo "$lv\n";
	
		$URI_GETPLAYERSTATUS = "http://live.nicovideo.jp/api/getplayerstatus?v=";

		$contents = GET($URI_GETPLAYERSTATUS . $lv, $user_session);
	
		$xml = simplexml_load_string($contents);
	
		#print_r($xml);

		$url =  $xml->rtmp->url;
		$url = str_replace("rtmp:rtmp", "rtmp", $url);
		$id = $xml->stream->id;
		$ticket = $xml->rtmp->ticket;
		$cont = $xml->stream->contents_list->contents;
		$cont = str_replace("rtmp:rtmp", "rtmp", $cont);


	 	echo $cmd = "rtmpdump -vr \"$url/$id\" -C S:\"$ticket\" -N \"$cont\" -p \"http://live.nicovideo.jp/watch/$lv\" -o - | vlc -";

		passthru($cmd);
		function Login($id, $password){
			$URI_LOGIN = "https://secure.nicovideo.jp/secure/login?site=niconico";

			$data = array(
				"mail" => $id,
				"password" => $password,
				"next_url" => ""
			);
			$data = http_build_query($data, "", "&");

			//header
			$header = array(
				"Content-Type: application/x-www-form-urlencoded",
				"Content-Length: ".strlen($data)
			);

			$context = array(
				"http" => array(
				"method"  => "POST",
				"header"  => implode("\r\n", $header),
				"content" => $data
				)
			);

			file_get_contents($URI_LOGIN, false, stream_context_create($context));

			$user_session = "";
			foreach ($http_response_header as $val) {
			  if ( preg_match("/Set-Cookie: (user_session=user_session.*)/u", $val, $match) ) {
				$user_session =  $match[1] . "\n";
			  }
			}


			file_put_contents("user_session", $user_session);
		}


		function GET($url, $session){
			$options = array(
				'http'=>array(
				'method'=>'GET',
				'header'=>"User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)\r\n".
					    "Accept-language: ja\r\n".
						"Cookie: $session\r\n"
				)
			);

			$context = stream_context_create( $options );
			return file_get_contents( $url , FALSE, $context );
		}

	?>

