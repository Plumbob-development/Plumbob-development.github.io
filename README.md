# Plumbob-development.github.io
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="referrer" content="strict-origin-when-cross-origin">
<title>The Anchor: club player</title>
</head>
<body>
<!--
  The Anchor's YouTube player (RivermereRP, rp_work's DJ). Garry's Mod opens this page in each player's game
  as dj_player.html?v=<video id>&t=<seconds>&play=<1 or 0>&vol=<0-100>; the script below is the same one the
  game builds into a plain page until this one is hosted (cl_dj_youtube.lua, PLAYER_JS). Host it on https (for
  example GitHub Pages), then set YouTubePage in rp_work's sh_config.lua to its address.
-->
<script>
(function () {
  if (window.rpdj && window.rpdj.go) return;
  var R = window.rpdj = {};
  function L(s) { console.log('RPDJ|' + s); }
  R.want = { vol: 0, play: false };
  R.now = function () {
    try { return R.p.getCurrentTime().toFixed(2) + ' ' + R.p.getPlayerState() + ' ' + (R.p.getDuration() || 0).toFixed(1); }
    catch (e) { return '0 -1 0'; }
  };
  R.go = function (o) {
    L('ua ' + navigator.userAgent);
    if (o.inject) { document.body.innerHTML = ''; }
    var st = document.createElement('style');
    st.textContent = 'html,body{margin:0;padding:0;width:100%;height:100%;background:#000;overflow:hidden}' +
      '#rpdj{position:fixed;left:0;top:0;width:100%;height:100%}';
    document.head.appendChild(st);
    var d = document.createElement('div');
    d.id = 'rpdj';
    document.body.appendChild(d);
    R.want.vol = o.vol;
    R.want.play = o.play;
    window.onYouTubeIframeAPIReady = function () {
      R.p = new YT.Player('rpdj', {
        width: '100%', height: '100%', videoId: o.vid,
        playerVars: { autoplay: o.play ? 1 : 0, start: Math.floor(o.start || 0), playsinline: 1, controls: 0, disablekb: 1,
          fs: 0, rel: 0, iv_load_policy: 3, origin: location.origin },
        events: {
          onReady: function () {
            R.ready = true;
            R.p.setVolume(R.want.vol);
            if ((o.start || 0) > 0.5) { R.p.seekTo(o.start, true); }
            if (R.want.play) { R.p.playVideo(); }
            L('ready ' + (R.p.getDuration() || 0));
          },
          onStateChange: function (e) {
            if (e.data === 1) { try { R.p.unloadModule('captions'); R.p.unloadModule('cc'); } catch (x) {} }
            L('state ' + e.data + ' ' + R.now());
          },
          onError: function (e) { L('err ' + e.data); },
          onAutoplayBlocked: function () { L('blocked'); }
        }
      });
    };
    var s = document.createElement('script');
    s.src = 'https://www.youtube.com/iframe_api';
    s.onerror = function () { L('err noapi'); };
    document.head.appendChild(s);
    setInterval(function () { if (R.ready) { L('t ' + R.now()); } }, 500);
  };
  R.vol = function (n) { R.want.vol = n; if (R.ready) { R.p.setVolume(n); } };
  R.seek = function (t) { if (R.ready) { R.p.seekTo(t, true); } };
  R.play = function () { R.want.play = true; if (R.ready) { R.p.playVideo(); } };
  R.pause = function () { R.want.play = false; if (R.ready) { R.p.pauseVideo(); } };
})();
</script>
<script>
(function () {
  var q = new URLSearchParams(location.search);
  var vid = q.get('v') || '';
  if (!/^[A-Za-z0-9_-]{11}$/.test(vid)) { console.log('RPDJ|err 2'); return; }
  var vol = parseInt(q.get('vol'), 10);
  rpdj.go({ vid: vid, start: parseFloat(q.get('t')) || 0, play: q.get('play') !== '0',
    vol: isNaN(vol) ? 0 : Math.max(0, Math.min(100, vol)) });
})();
</script>
</body>
</html>
