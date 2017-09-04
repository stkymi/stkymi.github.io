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
<script>plyr.setup();</script>

<!-- 嵌入播放器开始 -->
<div id="mediaplayer">JW Player goes here</div>
<script type="text/javascript">
		jwplayer("mediaplayer").setup({
	
			file: "http://l.symi.ml/Mr.Mr.mp4",
                        width: "100%",
                        aspectratio: "16:9",
                        preload: 'metadata',
			skin: {
                           name: "vapor"
		}
		});
</script> 
<!-- 嵌入播放器结束 -->
