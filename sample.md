---
layout: page
title: Sample Page
permalink: /sample/
---


<div class="plyr">
    <video controls>
        <!-- Video files -->
        <source src="http://l.symi.ml/catallena.mp4" type="video/mp4">

    </video>
    
</div>
<script>plyr.setup();
var timer; var hidding = false; $(function () { $('.plyr').mousemove(function () { if(hidding){ hidding = false; return; } if (timer) { clearTimeout(timer); timer = 0; } $('.plyr__video-wrapper').css({ cursor: 'pointer' }); timer = setTimeout(function () { hidding = true; $('.plyr__video-wrapper').css({ cursor: 'none' }); }, 2000) }); });
</script>

<!-- 嵌入播放器开始 -->
<div id="mediaplayer">JW Player goes here</div>
<script type="text/javascript">
		jwplayer("mediaplayer").setup({
	
			file: "http://l.symi.ml/Mr.Mr.mp4",
                        width: "100%",
                        aspectratio: "16:9",
                        preload: 'auto',
			skin: {
                           name: "vapor"
		}
		});
</script> 
<!-- 嵌入播放器结束 -->
