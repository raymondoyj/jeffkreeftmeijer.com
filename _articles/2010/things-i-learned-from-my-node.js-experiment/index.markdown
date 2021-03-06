<p>Last week, I published an article about <a href="http://jeffkreeftmeijer.com/2010/experimenting-with-node-js">my very first experiment</a> with <a href="http://nodejs.org">Node.js</a>. Being completely new to the whole thing, I decided to create a simple demo with web sockets.</p>
<p>As you can read in last week&#8217;s article, I created a page that checks your mouse cursor&#8217;s location and sends it to the web socket. The web socket broadcasts your mouse location to every other user on the page, so everyone will see your cursor moving around. That means you will see everybody else&#8217;s cursor moving around on your screen as well.</p>
<p><object width="440" height="308"><br><param name="allowfullscreen" value="true">
<br><param name="allowscriptaccess" value="always">
<br><param name="movie" value="http://vimeo.com/moogaloop.swf?clip_id=13805413&amp;server=vimeo.com&amp;show_title=0&amp;show_byline=0&amp;show_portrait=0&amp;color=ff9933&amp;fullscreen=1">
<br><embed src="http://vimeo.com/moogaloop.swf?clip_id=13805413&amp;server=vimeo.com&amp;show_title=1&amp;show_byline=1&amp;show_portrait=0&amp;color=ff9933&amp;fullscreen=1" type="application/x-shockwave-flash" allowfullscreen="true" allowscriptaccess="always" width="440" height="308"></embed><br></object></p>
<p>After letting my experiment run for a week, I learned a lot and improved the code. In this article I&#8217;ll go over some things I learned and how I fixed some issues.</p>
<p>Oh, and I&#8217;ve updated <a href="http://gist.github.com/488562">the Gist</a>, in case you want to use the code.</p>
<h3>People love fancy tricks with web sockets</h3>
<p>After posting the article, it jumped to the <a href="http://news.ycombinator.com/item?id=1548321">Hacker News</a> front page within minutes and stayed there for over 26 hours, got <a href="http://tweetmeme.com/story/1782313910/experimenting-with-nodejs-jeff-kreeftmeijer">retweeted like crazy</a> and got featured on <a href="http://5by5.tv/devshow/17">episode #17 of the Dev show</a>.</p>
<p>A lot of people commented or sent me replies via twitter and it was fun to see how cursors chased each other around, felt the urge to <a href="http://twitter.com/jnunemaker/status/19599899506">hump</a> or <a href="http://twitter.com/evilhackerdude/status/19687401793">bang</a> other cursors, <a href="http://www.reddit.com/r/programming/comments/ctvtz/experimenting_with_nodejs/c0v8243">draw shapes</a>, or make sure nobody could read the text by constantly hovering over it.</p>
<p>Thanks for the enthusiasm everybody, it was a lot of fun.</p>
<h3>Node.js handled everything perfectly</h3>
<p>Since this was my first experiment with Node.js, I expected cursors to start moving slow and the server to eventually crash. Especially because of this unexpected stream of visitors.</p>
<p>I added a rate limit of forty milliseconds &#8212; <code>mousemove()</code> gets fired <em>a lot</em> when you move your mouse &#8212; to take some load off the server and I got <a href="http://god.rubyforge.org/">God</a> to monitor it all and restart the server if it crashed.</p>
<p>However, except for some mistakes I made which caused the server to crash when somebody injected some javascript, Node.js handled everything perfectly. On a 256 MB <span class="caps">VPS</span>. Amazing.</p>
<h3>You can fall back to flash</h3>
<p>I posted a big notice above my article telling everyone to use Safari or Chrome, since they have support for web sockets. I didn&#8217;t know you could fall back to flash at the time.</p>
<p>Right now, the example runs on <a href="http://twitter.com/rauchg" title="Guillermo Rauch">@rauchg</a>&#8216;s <a href="http://github.com/LearnBoost/Socket.IO-node">Socket.IO</a> instead of <a href="http://github.com/miksago/node-websocket-server">web-socket-server</a>, which automatically switches to flash when your browser doesn&#8217;t support web sockets.</p>
<p>When you do this for yourself, please start the server with <code>sudo</code>. I couldn&#8217;t get it to work properly and eventually got help from Guillermo himself in the #node.js <span class="caps">IRC</span> channel.</p>
<p>The <a href="http://jeffkreeftmeijer.com/2010/experimenting-with-node-js/">demo</a> is still online, if you want to take it for a spin in your web socket-less browser.</p>
<h3>Injecting data into web sockets is easy</h3>
<p>Since I needed to send cursor locations on <code>mousemove()</code>, I had to create a connection to the web socket using Javascipt. This made it quite easy to send anything you want by injecting some Javascript into the page and calling <code>conn.send</code>. The web socket server would simply broadcast it to the other clients.</p>
<p>I was amazed to see <a href="http://twitter.com/einaros" title="Einar Otto Stangvik">@einaros</a> took the time to inject multiple cursors into the page and rotate them in 3D space. He even made <a href="http://www.youtube.com/watch?v=jULOA7mSOac">a video</a>.</p>
<p>How would we prevent users from doing something like this? I haven&#8217;t come up with a solution yet, since I think it&#8217;s impossible to find out if the messages get sent on <code>mousemove()</code> or if they&#8217;re simply called using a javascript console. I&#8217;d love to hear your thoughts on this.</p>
<h3>I should have sanitized the messages</h3>
<p>Although I ignored evil messages on the client side &#8212; I checked if the &#8220;action&#8221; was &#8220;move&#8221; or &#8220;close&#8221; &#8212; the Node.js server would still broadcast any <span class="caps">JSON</span> message it received. That meant injecting something like this would result in it being sent to every other client:</p>
<div class="highlight">
<pre><code class="javascript">  <span class="nx">conn</span><span class="p">.</span><span class="nx">send</span><span class="p">({</span><span class="s1">'injected!'</span><span class="o">:</span> <span class="s1">'haha!'</span><span class="p">});</span>
</code></pre>
</div>
<p>Again, nothing would happen if a client would receive this message because it wouldn&#8217;t contain anything it would understand. Still, not very nice to just broadcast this to everyone.</p>
<p>Right now, the message listener catches syntax errors when trying to parse the message and checks if the action is &#8220;close&#8221; or &#8220;move&#8221;:</p>
<div class="highlight">
<pre><code class="javascript"><span class="nx">server</span><span class="p">.</span><span class="nx">addListener</span><span class="p">(</span><span class="s2">"connection"</span><span class="p">,</span> <span class="kd">function</span><span class="p">(</span><span class="nx">conn</span><span class="p">){</span>
  <span class="nx">conn</span><span class="p">.</span><span class="nx">addListener</span><span class="p">(</span><span class="s2">"message"</span><span class="p">,</span> <span class="kd">function</span><span class="p">(</span><span class="nx">message</span><span class="p">){</span>
    
    <span class="k">try</span> <span class="p">{</span>
      <span class="nx">request</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nx">parse</span><span class="p">(</span><span class="nx">message</span><span class="p">);</span>
    <span class="p">}</span> <span class="k">catch</span> <span class="p">(</span><span class="nx">SyntaxError</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">log</span><span class="p">(</span><span class="s1">'Invalid JSON:'</span><span class="p">);</span>
      <span class="nx">log</span><span class="p">(</span><span class="nx">message</span><span class="p">);</span>
      <span class="k">return</span> <span class="kc">false</span><span class="p">;</span>
    <span class="p">}</span>
    
    <span class="k">if</span><span class="p">(</span><span class="nx">request</span><span class="p">.</span><span class="nx">action</span> <span class="o">!=</span> <span class="s1">'close'</span> <span class="o">&amp;&amp;</span> <span class="nx">request</span><span class="p">.</span><span class="nx">action</span> <span class="o">!=</span> <span class="s1">'move'</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">log</span><span class="p">(</span><span class="s1">'Ivalid request:'</span><span class="p">);</span>
      <span class="nx">log</span><span class="p">(</span><span class="nx">message</span><span class="p">);</span>
      <span class="k">return</span> <span class="kc">false</span><span class="p">;</span>
    <span class="p">}</span>
    
    <span class="nx">request</span><span class="p">.</span><span class="nx">id</span> <span class="o">=</span> <span class="nx">conn</span><span class="p">.</span><span class="nx">id</span>
    <span class="nx">conn</span><span class="p">.</span><span class="nx">broadcast</span><span class="p">(</span><span class="nx">JSON</span><span class="p">.</span><span class="nx">stringify</span><span class="p">(</span><span class="nx">request</span><span class="p">));</span>    
  <span class="p">});</span>
<span class="p">});</span>
</code></pre>
</div>
<h3>More?</h3>
<p>If you have any more feedback on the code sample, be sure to let me know. <a href="http://gist.github.com/488562">The gist</a> is updated and be sure to check out the <a href="http://jeffkreeftmeijer.com/2010/experimenting-with-node-js/">new and improved version of the experiment</a>.</p>