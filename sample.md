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
    <div class="plyr">
    <button type="button" class="btn js-play">Play</button>
    <button type="button" class="btn js-pause">Pause</button>
    <button type="button" class="btn js-stop">Stop</button>
    <button type="button" class="btn js-rewind">Rewind</button>
    <button type="button" class="btn js-forward">Forward</button>
  </div>
</div>
<script>plyr.setup();</script>

<!-- 嵌入播放器开始 -->
<div id="mediaplayer">JW Player goes here</div>
<script type="text/javascript">
		jwplayer("mediaplayer").setup({
	
			file: "http://l.symi.ml/Mr.Mr.mp4",
                        width: "100%",
                        aspectratio: "16:9",
			skin: {
                           name: "vapor"
		}
		});
</script> 
<!-- 嵌入播放器结束 -->
