// 見ないでぇ…

// 再現可能な乱数の生成
class Random {
  constructor(seed = 48415698) {
    this.x = 198378468;
    this.y = 987194765;
    this.z = 569817352;
    this.w = seed;
  }
  
  // XorShift
  next() {
    let t;
 
    t = this.x ^ (this.x << 11);
    this.x = this.y; this.y = this.z; this.z = this.w;
    return this.w = (this.w ^ (this.w >>> 19)) ^ (t ^ (t >>> 8)); 
  }
  
  // min以上max以下の乱数を生成する
  nextInt(min, max) {
    const r = Math.abs(this.next());
    return min + (r % (max + 1 - min));
  }
}


var msgColours = {
  "success": "#0f8c1f",
  "error": "red",
  "general": "yellow",
};

// 定数
var SYSMESSAGE = '#SYSMESSAGE';
var ADMINNAME = 'korehaTaihen';

// 汎用関数
function replaceAll(msg, substr, newstr)
{
  return msg.split(substr).join(newstr);
}

var isPlaying = true; // this is an assumption, and should be overriden on load anyway
var onPlayerStateChange = function(status) {
  // other states available, https://developers.google.com/youtube/iframe_api_reference#Examples
  if (status === 1) { // Playing
    isPlaying = true;
  } else if (status === 2) { // Paused
    isPlaying = false;
  }
};

var addBotMsg = function(msg, colour) {
  var element = document.createElement('div');
  element.classList.add('server-msg-reconnect'); // Spoofs a server message
  element.style.color = colour;
  element.style.border = "1px solid " + colour;
  element.innerText = msg;
  document.querySelector('#messagebuffer').appendChild(element);
  document.querySelector('#messagebuffer').scrollTop = document.querySelector('#messagebuffer').scrollHeight;
};

var sendMsg = function(msg) {
  document.querySelector('#chatline').value  = msg;
  // Due to a bug outside of my control, this is how a enter press must be dispatched
  var enterEvent = new Event('keydown');
  enterEvent.keyCode = 13;
  document.querySelector('#chatline').dispatchEvent(enterEvent);
};

var addNameToTarget = function(targ) {
  if (targ.childNodes.length < 3 && targ.className.indexOf("server-msg") < 0 && targ.className.indexOf("\\$server\\$") < 0) {
    var span = document.createElement("SPAN");
    span.innerHTML = "";

    targ.insertBefore(span, targ.childNodes[1]);
  }
};

/*
  ChildNodesに入っている情報
  0:タイムスタンプ
  1:ユーザー名
  2:発言
*/

// 発言内に特定コマンドを文字列が入っているかチェック
// 入っていた場合、htmlの置き換えを行います
var checkForEmoteCommands = function(targ, isInit) {
  if (isInit === undefined) isInit = false;

  // Check vanilla message
  if (targ.childNodes[2] !== undefined) {

    // 画像URLを画像化
    imgCommandFunc(targ);

    // imgタグの編集系
    //置き換えが無くなるまで繰り返す
    let cmd_max = 10;
    while(true)
    {
      var suc = false;
      if(ImgSetCmdClass(targ, "big", cmd_max, true)) suc = true;
      if(ImgSetCmdClass(targ, "small", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "mr", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "mrbig", cmd_max, true)) suc = true;
      if(ImgSetCmdClass(targ, "rt90", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "rt180", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "rt-90", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "nega", cmd_max)) suc = true;

      if(ImgSetCmdClass(targ, "spin", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "damage", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "jump", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "hedoban", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "attackr", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "attackl", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "buruburu", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "knock", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "zoomin", cmd_max, true)) suc = true;
      if(ImgSetCmdClass(targ, "zoomout", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "kurukuru", cmd_max)) suc = true;
      if(ImgSetCmdClass(targ, "dareda", cmd_max)) suc = true;
      if(suc == false) break;
    }
  }
};

var ImgSetCmdClass = function(targ, cmd, cmdMax, isTrim = false)
{
  var rep = false;

  var ccmd = "!" + cmd + " "; 
  var html = targ.childNodes[2].innerHTML;

  var ind = html.indexOf(ccmd);
  if(ind == -1) return rep;

  // imgのチェック
  if(ImgSetCmdClassRep(targ, cmd, cmdMax, "img", isTrim) === true) rep = true;
  if(ImgSetCmdClassRep(targ, cmd, cmdMax, "div", isTrim) === true) rep = true;
  return rep;
}

var ImgSetCmdClassRep = function(targ, cmd, cmdMax, elementName, isTrim)
{
  var repFlg = false;
  var html = targ.childNodes[2].innerHTML;
  var ccmd = "!" + cmd + " <" + elementName + " "; 
  var ind = html.indexOf(ccmd);
  if(ind != -1)
  {
    var rep = "<" + elementName + " id='replacer' ";
    html = html.slice(0, ind) + rep + html.slice(ind + ccmd.length);
    targ.childNodes[2].innerHTML = html;
    
    var ele = document.getElementById("replacer");
    if(ele != null)
    {
      //ele.id = "";
      ele.removeAttribute("id");

      // エモートコマンド数が限界を超えている場合無効
      let cmdcnt = ele.getAttribute("cmdcnt");
      if(cmdcnt === "" || cmdcnt === undefined || cmdcnt === null) cmdcnt = 0;
      if(cmdcnt >= cmdMax-1) repFlg = true;
      else
      {
        repFlg = true;
        cmdcnt++;
        ele.removeAttribute("cmdcnt");

        if(isTrim == false)
        {
          //ele.outerHTML = "<div class='" + cmd + " custom-emote channel-emote'>" + ele.outerHTML + "</div>";    
          ele.outerHTML = "<div class='" + cmd + " custom-emote'" + " cmdcnt='" + cmdcnt + "'>" + ele.outerHTML + "</div>";    
        }
        else
        {
          //ele.classList.add("maximize");
          ele.classList.add(cmd);
          ele.classList.add("maximize");
          //ele.outerHTML = "<div class='trim custom-emote'" + " cmdcnt='" + cmdcnt + "'>" + "<div class='" + cmd + "'>" + ele.outerHTML + "</div></div>";
          ele.outerHTML = "<div class='trim custom-emote'" + " cmdcnt='" + cmdcnt + "'>" + ele.outerHTML + "</div></div>";
        }
      }
    }
    let h = ele.outerHTML;
  }
  return repFlg;
}

var setOnClickEvent = function(targ, isInit) {
  if (isInit === undefined) isInit = false;

  var imgs = targ.getElementsByClassName("channel-emote");//targ.querySelectorAll('img');
  for (let i = 0; i < imgs.length; i++) {
    imgs[i].onclick = function() {
      var str = imgs[i].title;

      var chatline = document.getElementById("chatline");
      if(chatline.value.length !== 0 && chatline.value.slice(-1) !== ' ') chatline.value += ' ';
      chatline.value += str;
      chatline.focus();
    }

    imgs[i].style = "cursor : pointer;";
  }
};

var imgCommandFunc = function(targ)
{
  // !imgをURL画像に変換します
  var text = targ.childNodes[2].innerText;


  // a要素を探します
  for (let i = 0; i < targ.childNodes[2].childNodes.length; i++) {
    var node = targ.childNodes[2].childNodes[i];
    let nn = node.nodeName;
    if(nn.toLowerCase() === "a")
    {
      let url = node.innerText;
      // URLであるか判定
      if(url.slice(0,8) === "https://" || url.slice(0,7) === "http://") // http://
      {
        // 画像であるかチェック
        let isImg = false;
        if(url.slice(-4).toLowerCase() === ".gif") isImg = true;
        else if(url.slice(-4).toLowerCase() === ".png") isImg = true;
        else if(url.slice(-4).toLowerCase() === ".jpg") isImg = true;
        else if(url.slice(-5).toLowerCase() === ".jpeg") isImg = true;

        if(isImg == true)
        {
          // 該当nodeそのものを置き換えれば完成
          InsertAfterImage(targ.childNodes[2], node, url);
          node.remove();
        }
      }
    }
  }
}

var InsertAfterImage = function(ele, refEle, url) {
  var buffer = document.querySelector('#messagebuffer');

  var image = new Image();
  image.onload = function() { buffer.scrollTop = buffer.scrollHeight; };
  image.onerror = function() { image.onerror = ""; image.src = "https:\/\/i.imgur.com\/VWzRomm.png"; };
  image.src = url;

  var imgElement = document.createElement('img');
  imgElement.src = image.src;
  imgElement.style["max-width"] = "150px";
  imgElement.style["max-height"] = "100px";
  imgElement.style.cursor = "zoom-in";
  imgElement.classList.add('user-img');
  imgElement.style["border"] = "solid 2px #ffffff";
  imgElement.style["border-radius"] = "0.5em";
  ele.insertBefore(imgElement, refEle);
  //targ.childNodes[2].remove();

  var imgDiv = document.createElement('div');
  imgDiv.style.position = "fixed";
  imgDiv.style.top = 0;
  imgDiv.style.left = 0;
  imgDiv.style.width = document.body.offsetWidth + "px";
  imgDiv.style.height = document.body.offsetHeight + "px";
  imgDiv.style["background-color"] = "rgba(23, 23, 23, 0.75)";
  imgDiv.style["z-index"] = 2000; // Has to be above the fixed header
  imgDiv.style.visibility = "hidden";
  imgDiv.style.cursor = "zoom-out";
  var imgPreview = document.createElement('img');
  imgPreview.src = image.src;
  imgPreview.style.visibility = "hidden";
  imgPreview.style.position = "fixed";
  imgPreview.style.top = "0px";
  imgPreview.style.left = "0px";
  imgPreview.style["z-index"] = 25;
  imgPreview.style.cursor = "zoom-out";

  imgDiv.appendChild(imgPreview);
  ele.insertBefore(imgDiv, refEle);

  var clickShow = function(e) {
    // Must be called here as offsetWidth/Height may change
    imgDiv.style.width = window.innerWidth + "px";
    imgDiv.style.height = window.innerHeight + "px";
    imgDiv.style.visibility = "visible";

    imgPreview.style["max-width"] = window.innerWidth*0.8 + "px";
    imgPreview.style["max-height"] = window.innerHeight*0.98 + "px";
    imgPreview.style.top = ((window.innerHeight - imgPreview.height) / 2) + "px";
    imgPreview.style.left = ((window.innerWidth - imgPreview.width) / 2) + "px";
    imgPreview.style.visibility = "visible";
  };

  var clickHide = function(e) {
    imgPreview.style.visibility = "hidden";
    imgDiv.style.visibility = "hidden";
  };

  imgElement.addEventListener('click', clickShow);

  imgDiv.addEventListener('click',  clickHide);
  imgPreview.addEventListener('click',  clickHide);

  window.onresize = function() {
    imgDiv.style.width = window.innerWidth + "px";
    imgDiv.style.height = window.innerHeight + "px";

    imgPreview.style["max-width"] = window.innerWidth*0.8 + "px";
    imgPreview.style["max-height"] = window.innerHeight*0.98 + "px";
    imgPreview.style.top = ((window.innerHeight - imgPreview.height) / 2) + "px";
    imgPreview.style.left = ((window.innerWidth - imgPreview.width) / 2) + "px";
  };
};

// 先頭に特定文字列が入っているかチェック
// 入っていた場合、その発言をコマンドとして扱います
var checkForHeadCommands = function(targ, isInit) {
  if (isInit === undefined) isInit = false;
  
  // Check vanilla message
  if (targ.childNodes[2] !== undefined) {
    let cmd = SYSMESSAGE + " ";
    if(targ.childNodes[2].innerHTML.slice( 0, 12 ) === cmd)
    {
      // システムメッセージの表示
      var ts = targ.childNodes[0].innerHTML;
      var username = targ.className.substring("chat-msg-".length, targ.className.length);
      var text = targ.childNodes[2].innerHTML;

      targ.childNodes[0].innerHTML = "";
      targ.childNodes[1].innerHTML = "";
      targ.childNodes[2].innerHTML = targ.childNodes[2].innerHTML.replace(cmd, "").replace("{0}", ts).replace("{1}", username);
      targ.childNodes[2].style = 'font-weight: 550; font-family: "arial"; color: #80f080;';

      return true;
    }
  }
};

// --------------------------------------------------------------------------
var main = function() {
  addBotMsg('Version 1.4.1.3', msgColours.error);

  var msgTarget = document.getElementById("messagebuffer");

  var msgList = msgTarget.getElementsByTagName("div");
  for (var i = 0; i < msgList.length; i++) {
    addNameToTarget(msgList[i]);
  }

  // Inits variables for tab visibility and message notifications
  // タブの可視性とメッセージ通知のための整数変数
  if (document.hidden !== undefined) {
    hidden = 'hidden';
    visibilityEvent = 'visibilitychange';
  } else if (document.msHidden !== undefined) {
    hidden = 'msHidden';
    visibilityEvent = 'msvisibilitychange';
  } else if (document.webkitHidden !== undefined) {
    hidden = 'webkitHidden';
    visibilityEvent = 'webkitvisibilitychange';
  }
  
  // 新しいメッセージが入力された際の処理
  var msgObserver = new MutationObserver(function(mutations) {
    for (let i = 0; i < mutations.length; i++) {
      var newNodes = mutations[i].addedNodes;
      for (let k = 0; k < newNodes.length; k++) {
        addNameToTarget(newNodes[k]);
        if(!checkForHeadCommands(newNodes[k], false))
        {
          checkForEmoteCommands(newNodes[k]);
          setOnClickEvent(newNodes[k]);
        }
      }
    }
  });

  // configuration of the observer:
  var msgConfig = {childList : true};

  // 過去メッセージの全チェック
  for (let i = msgTarget.childNodes.length-1; i >= 0; i--) {
    if(!checkForHeadCommands(msgTarget.childNodes[i], true))
    {
      checkForEmoteCommands(msgTarget.childNodes[i], true);
      setOnClickEvent(msgTarget.childNodes[i], true);
    }
    scrollChat();
  }

  // pass in the target node, as well as the observer options
  // ターゲットノードとオブザーバオプションを渡す
  msgObserver.observe(msgTarget, msgConfig);

  // Remove "advertisement" if it exists - allows adding of potentially updated one if code changes
  // 「広告」があれば削除
  if (document.querySelector('a[href="https://github.com/BranchofLight/CyTube-Room-Script"]') !== null) {
    document.querySelector('a[href="https://github.com/BranchofLight/CyTube-Room-Script"]').remove();
  }

  if (document.querySelector('.credit').querySelector('span') !== null) {
    document.querySelector('.credit').querySelector('span').remove();
  }

  // Add any custom CSS
  // Used over cytube CSS editor so you only have to update one file
  // カスタムcss追加
  var css = `
  .custom-emote {
    display: inline-block;
  }
  .maximize
  {
    width: 100px;
    height: 100px;
    object-fit: contain;
  }
  .trim {
    vertical-align: middle;
    overflow: hidden;
    object-fit: none;
  }
  .big {
    transform: scale(2, 2);
  }
  .small {
    transform: scale(0.5, 0.5);
  }
  .mr {
    transform-origin: center center;
    transform: scale(-1, 1);
  }
  .mrbig {
    transform-origin: center center;
    transform: scale(-1.5, 1.5);
  }
  .rt90 {
    transform-origin: center center;
    transform: rotate(90deg);
  }
  .rt180 {
    transform-origin: center center;
    transform: rotate(180deg);
  }
  .rt-90 {
    transform-origin: center center;
    transform: rotate(270deg);
  }
  @-moz-keyframes spin {
    from { -moz-transform: rotate(0deg); }
    to { -moz-transform: rotate(359deg); }
  }
  @-webkit-keyframes spin {
      from { -webkit-transform: rotate(0deg); }
      to { -webkit-transform: rotate(359deg); }
  }
  .nega {
    filter: invert(100%);
  }

  @keyframes spin {
      from {transform:rotate(0deg);}
      to {transform:rotate(359deg);}
  }
  .spin {
    animation-duration: 2s;
    animation-name: spin;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
  }
  @keyframes damage {
    0%, 49% {filter: hue-rotate(0deg); opacity:1;}
    50%, 100% {filter: hue-rotate(0deg); opacity:0.5;}
  }
  .damage {
    animation-duration: 0.2s;
    animation-name: damage;
    animation-timing-function: steps(2, end);
    animation-iteration-count: infinite;
  }
  @keyframes jump {
    0%, 25% {transform:  translate(0px, 0px);}
    38.5%, 86.5% {transform:  translate(0px, -50px);}
    50%, 74% {transform:  translate(0px, -88px);}
    58%, 66% {transform:  translate(0px, -96px);}
    62% {transform:  translate(0px, -100px);}
    100% {transform:  translate(0px, 0px);}
  }
  .jump {
    animation-duration: 1s;
    animation-name: jump;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
  }
  @keyframes hedoban {
    100% {transform:  scale(1, 1);}
    50% {transform:  scale(1.05, 0.95);}
    100% {transform:  scale(1, 1);}
  }
  .hedoban {
    animation-duration: 0.333s;
    animation-name: hedoban;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
    transform-origin: center bottom;
  }
  @keyframes attackr {
    0%, 25% {transform:  translate(0px, 0px);}
    30% {transform:  translate(-0.8px, -2px);}
    38.75% {transform:  translate(-1.6px, -7px);}
    43.75% {transform:  translate(-2.4px, -10px);}
    48.75% {transform:  translate(-3.2px, -7px);}
    57.5%  {transform:  translate(-4.0px, -2px);}
    62.5%  {transform:  translate(-5.0px, 0px);}
    81.25% {transform:  translate(20px, 0px);}
    100% {transform:  translate(0px, 0px);}
  }
  .attackr {
    animation-duration: 0.75s;
    animation-name: attackr;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
  }
  @keyframes attackl {
    0%, 25% {transform:  translate(0px, 0px);}
    30% {transform:  translate(0.8px, -2px);}
    38.75% {transform:  translate(1.6px, -7px);}
    43.75% {transform:  translate(2.4px, -10px);}
    48.75% {transform:  translate(3.2px, -7px);}
    57.5%  {transform:  translate(4.0px, -2px);}
    62.5%  {transform:  translate(5.0px, 0px);}
    81.25% {transform:  translate(-20px, 0px);}
    100% {transform:  translate(0px, 0px);}
  }
  .attackl {
    animation-duration: 0.75s;
    animation-name: attackl;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
  }
  @keyframes buruburu {
    0% {transform: translate(0px, 0px) rotateZ(0deg)}
    25% {transform: translate(2px, 2px) rotateZ(1deg)}
    50% {transform: translate(0px, 2px) rotateZ(0deg)}
    75% {transform: translate(2px, 0px) rotateZ(-1deg)}
    100% {transform: translate(0px, 0px) rotateZ(0deg)}
  }
  .buruburu {
    animation-duration: 0.1s;
    animation-name: buruburu;
    animation-timing-function: steps(6, end);
    animation-iteration-count: infinite;
  }
  @keyframes knock {
    0%  {transform: translate(0px, 0px);}
    19% {transform: translate(38px, -50px);}
    34% {transform: translate(68px, -88px);}
    45% {transform: translate(90px, -96px);}
    50% {transform: translate(100px, -100px);}
    55% {transform: translate(110px, -96px);}
    66% {transform: translate(132px -88px);}
    81% {transform: translate(162px, -50px);}
    100% {transform: translate(200px, 0px);}
  }
  .knock {
    animation-duration: 0.75s;
    animation-name: knock;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
    animation-delay: 0.6s;
  }
  @keyframes zoomin {
    0% {transform:  scale(0, 0);}
    100% {transform:  scale(2, 2);}
  }
  .zoomin {
    animation-duration: 1s;
    animation-name: zoomin;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
    transform-origin: center center;
  }
  @keyframes zoomout {
    0% {transform:  scale(1, 1);}
    100% {transform:  scale(0, 0);}
  }
  .zoomout {
    animation-duration: 1s;
    animation-name: zoomout;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
    transform-origin: center center;
  }
  @keyframes kurukuru {
    0% {transform:  scale(1, 1);}
    50% {transform:  scale(-1, 1);}
    100% {transform:  scale(1, 1);}
  }
  .kurukuru {
    animation-duration: 0.5s;
    animation-name: kurukuru;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
    transform-origin: center center;
  }
  .dareda {
    filter: brightness(0%);
  }
  .dareda:hover {
    filter: brightness(100%);
  }
  `;

  var cssTag = document.createElement('style');
  if (cssTag.styleSheet) {
    cssTag.styleSheet.cssText = css;
  } else {
    cssTag.appendChild(document.createTextNode(css));
  }

  document.querySelector('head').appendChild(cssTag);
  window.addEventListener('message', function(e) {
    var data;
    try
    {
      data = JSON.parse(e.data);
    }
    catch
    {
      return;
    }
    if (data.event === 'onStateChange') {
      onPlayerStateChange(data.info);
    }
  });
}();

(() => {
  // css定義
  var css = `.channel-emote {
    max-width: 100px;
    max-height: 100px;
    }`
  var cssTag = document.createElement('style');
  if (cssTag.styleSheet) {
    cssTag.styleSheet.cssText = css;
  } else {
    cssTag.appendChild(document.createTextNode(css));
  }
  var style = document.querySelector('head').appendChild(cssTag);
  
  // クリックの処理
  $('<span class="label label-default pull-right pointer" id="small-emo" title="エモートを縮小">E3</span>').insertBefore("#emoteflair2").on("click", function(event) {
    // スイッチの切り替え
    if (style.disabled == true) style.disabled = false;
    else style.disabled = true;
    changeSmallEmoButtonVisual(style.disabled);
  });


  var changeSmallEmoButtonVisual = function(flag){
    var button = $("#small-emo")

    if(flag === false)
    {
      button.removeClass("label-default")
      button.addClass("label-success");
      setOpt(CHANNEL.name + "_small_emo", "1"); 
    }
    else 
    {
      button.removeClass("label-success");
      button.addClass("label-default")
      setOpt(CHANNEL.name + "_small_emo", "0"); 
    }
  }

  // 初期値の読み書き
  style.disabled = true;
  if (getOpt(CHANNEL.name + "_small_emo") !== "1" && getOpt(CHANNEL.name + "_small_emo") !== "0")
  {
    setOpt(CHANNEL.name + "_small_emo", "1"); 
  }
  if (getOpt(CHANNEL.name + "_small_emo") === "1") 
  {
    style.disabled = false;
    changeSmallEmoButtonVisual(style.disabled);
  }
})();

(() => {
  // 再生履歴の初期表示切替
  var VISIBLE_HISTORY = false; // デフォルトはtrue
  var cmd = "_showplayhist";
  if (getOpt(CHANNEL.name + cmd) !== "1" && getOpt(CHANNEL.name + cmd) !== "0")
  {
    setOpt(CHANNEL.name + cmd, "1"); 
  }
  if (getOpt(CHANNEL.name + cmd) === "0")
  {
    VISIBLE_HISTORY = false;
  }
  RGTVSetVisiblePlayhist(VISIBLE_HISTORY);
  
  $("#showplayhist").get(0).onclick = "";
  $("#showplayhist").click(function() {
    VISIBLE_HISTORY = !VISIBLE_HISTORY;
    if(VISIBLE_HISTORY === true) setOpt(CHANNEL.name + cmd, "1"); 
    else setOpt(CHANNEL.name + cmd, "0"); 

    RGTVSetVisiblePlayhist(VISIBLE_HISTORY);
  });

  function RGTVSetVisiblePlayhist(visible) {
    if(visible){
        $("#queue_history").parent().removeClass('collapse').addClass('in');
        $("#showplayhist").addClass("active");
    }else{
        $("#queue_history").parent().addClass('collapse').removeClass('in');
        $("#showplayhist").removeClass("active");
    }
  }
})();

(() => {
  // 動画キュー
  var VISIBLE_PLAYLIST = false; // デフォルトはtrue
  var cmd = "_showplaylist";
  if (getOpt(CHANNEL.name + cmd) !== "1" && getOpt(CHANNEL.name + cmd) !== "0")
  {
    setOpt(CHANNEL.name + cmd, "1"); 
  }
  if (getOpt(CHANNEL.name + cmd) === "0")
  {
    VISIBLE_PLAYLIST = false;
  }
  RGTVSetVisiblePlaylist(VISIBLE_PLAYLIST);
  
  $("#showplaylist").get(0).onclick = "";
  $("#showplaylist").click(function() {
    VISIBLE_PLAYLIST = !VISIBLE_PLAYLIST;
    if(VISIBLE_PLAYLIST === true) setOpt(CHANNEL.name + cmd, "1"); 
    else setOpt(CHANNEL.name + cmd, "0"); 

    RGTVSetVisiblePlaylist(VISIBLE_PLAYLIST);
  });

  function RGTVSetVisiblePlaylist(visible) {
    if(visible){
        $("#queue").parent().removeClass('collapse').addClass('in');
        $("#showplaylist").addClass("active");
    }else{
        $("#queue").parent().addClass('collapse').removeClass('in');
        $("#showplaylist").removeClass("active");
    }
  }
})();

// エモート一覧モーダル関連
(() => {
  var isShiftPress = false;

  $(document).on("keydown" , function(event){
    if(event.keyCode == 16) isShiftPress = true;
  });
  $(document).on("keyup" , function(event){
    if(event.keyCode == 16) isShiftPress = false;
    else if(window.EMOTELISTMODAL.css("display") == "block" && event.keyCode == 13)
    {
      window.EMOTELISTMODAL.modal("hide");
      chatline.focus();
    }
  });
  
  var oec = function RGTVOnEmoteClicked(emote){
    var val = chatline.value;
    if (!val) {
        chatline.value = emote.name;
    } else {
        if (!val.charAt(val.length - 1).match(/\s/)) {
            chatline.value += " ";
        }
        chatline.value += emote.name;
    }
    
    if(isShiftPress === false)
    {
      window.EMOTELISTMODAL.modal("hide");
      chatline.focus();
    }
  };

  window.EMOTELIST.emoteClickCallback = oec;
  window.EMOTELIST.loadPage(0);

  var str = document.createElement("div");
  str.innerHTML     = "※Shiftキーを押しながらクリックで連続入力できます";
  var modal = document.getElementsByClassName("emotelist-table")[0];
  modal.appendChild(str);
})();

// コマンド一覧ボタン
(() => {
  let emolist = document.getElementById("emotelist");

  // このチャンクはまじないみたいなものです。既存のエモート一覧に合わせてます
  var modal = document.createElement('div');
  modal.className = "modal fade";
  modal.id = "commandlist";
  modal.tabIndex = "-1";
  modal.role = "dialog";
  emolist.parentNode.insertBefore(modal, emolist.nextSibling); 
  let inner1 = document.createElement('div');
  inner1.className = "modal-dialog modal-dialog-nonfluid";
  modal.appendChild(inner1);
  let inner2 = document.createElement('div');
  inner2.className = "modal-content";
  inner1.appendChild(inner2);

  // ヘッダー
  let header = document.createElement('div');
  header.className = "modal-header";
  inner2.appendChild(header);
  let head = document.createElement('h4');
  head.textContent = "チャットコマンド一覧";
  header.appendChild(head);

  // ボディ
  let body = document.createElement('div');
  body.className = "modal-body";
  inner2.appendChild(body);

  // コマンドと説明
  var listCmd= [];
  listCmd.push({name:'!big エモート', desc:'エモートサイズが2倍になります。大きすぎる場合はトリミングされます。'});
  listCmd.push({name:'!small エモート', desc:'エモートサイズが半分になります。'});
  listCmd.push({name:'!mr エモート', desc:'エモートが左右反転します。'});
  listCmd.push({name:'!mrbig エモート', desc:'エモートサイズが2倍になり、左右反転します。大きすぎる場合はトリミングされます。'});
  listCmd.push({name:'!rt90 エモート', desc:'エモートが90度時計回りに回転します。'});
  listCmd.push({name:'!rt180 エモート', desc:'エモートが180度時計回りに回転します。'});
  listCmd.push({name:'!rt-90 エモート', desc:'エモートが90度半時計回りに回転します。'});
  listCmd.push({name:'!nega エモート', desc:'エモートの明暗、色相が反転します。'});
  listCmd.push({name:'!spin エモート', desc:'エモートが回転アニメーションします。'});
  listCmd.push({name:'!damage エモート', desc:'エモートが点滅します。'});
  listCmd.push({name:'!jump エモート', desc:'エモートがジャンプします。'});
  listCmd.push({name:'!hedoban エモート', desc:'エモートがリズミカルに動きます。'});
  listCmd.push({name:'!attackr エモート', desc:'エモートが右方向に攻撃します。'});
  listCmd.push({name:'!attackl エモート', desc:'エモートが左方向に攻撃します。'});
  listCmd.push({name:'!buruburu エモート', desc:'エモートがブルブルと震えます。'});
  listCmd.push({name:'!knock エモート', desc:'エモートが飛んでいきます。'});
  listCmd.push({name:'!zoomin エモート', desc:'エモートが膨れ上がります。'});
  listCmd.push({name:'!zoomout エモート', desc:'エモートがしぼんでいきます。'});
  listCmd.push({name:'!kurukuru エモート', desc:'エモートがくるくる回転します。'});
  listCmd.push({name:'!dareda エモート', desc:'エモートがシルエット状に表示されます。マウスを乗せると本来のエモートが表示されます。'});
  //listCmd.push({name:'!fd[メッセージ]', desc:'文字がドット絵になります。'});
  listCmd.push({name:'!emo[メッセージ]', desc:'中に入っている文字がエモート同様に扱われます。'});
  //listCmd.push({name:'!ozaki[メッセージ]', desc:'文字が吹き出し付きで表示されるぞ。'});
  //listCmd.push({name:'!fukidasia[メッセージ]', desc:'文字が吹き出し付きで表示されます。'});
  //listCmd.push({name:'!fukidasib[メッセージ]', desc:'文字が吹き出し付きで表示されます。'});
  
  let table = document.createElement('table');
  table.border = 1;
  table.bordercolor = "#ffffff";
  table.style = "border-style: solid; bordercolor:#ffffff; table-layout: auto;";
  body.appendChild(table);

  let tr = document.createElement('tr');
  tr.style = "background:#000000; color:#ffffff; font-weight:bold; text-align:center;";
  table.appendChild(tr);
  let th = document.createElement('th');
  th.textContent = "コマンド";
  th.width = "150px";
  tr.appendChild(th);
  th = document.createElement('th');
  th.textContent = "説明文";
  th.width = "400px";
  tr.appendChild(th);

  for (let i=0; i<listCmd.length; i++) 
  {
    let cmd = listCmd[i];
    let tr = document.createElement('tr');
    table.appendChild(tr);

    th = document.createElement('td');
    th.textContent =  cmd.name;
    tr.appendChild(th);
    th = document.createElement('td');
    th.textContent = cmd.desc;
    tr.appendChild(th);
  }

  // フッター
  let fotter = document.createElement('div');
  fotter.className = "modal-fotter";
  inner2.appendChild(fotter);

  var modal = $("#commandlist")
  $('<button class="btn btn-sm btn-default" id="commandlistbtn" style="float:right;">コマンド一覧</button>').insertAfter("#emotelistbtn").on("click", function(event) {
    // スイッチの切り替え
    modal.modal();
  });
})();


// ショートカット関連
(() => {
  let leftelement = document.getElementById("commandlist");

  // このチャンクはまじないみたいなものです。既存のエモート一覧に合わせてます
  var modal = document.createElement('div');
  modal.className = "modal fade";
  modal.id = "shortcutlist";
  modal.tabIndex = "-1";
  modal.role = "dialog";
  leftelement.parentNode.insertBefore(modal, leftelement.nextSibling); 
  let inner1 = document.createElement('div');
  inner1.className = "modal-dialog modal-dialog-nonfluid";
  modal.appendChild(inner1);
  let inner2 = document.createElement('div');
  inner2.className = "modal-content";
  inner1.appendChild(inner2);

  // ヘッダー
  let header = document.createElement('div');
  header.className = "modal-header";
  inner2.appendChild(header);
  let head = document.createElement('h4');
  head.textContent = "ショートカット一覧";
  header.appendChild(head);

  // ボディ
  let body = document.createElement('div');
  body.className = "modal-body";
  inner2.appendChild(body);

  // テーブル
  let table = document.createElement('table');
  table.width = "100%";
  body.appendChild(table);

  for (let i=0; i<10; i++) 
  {
    let tr = document.createElement('tr');
    tr.style = "height: 35px;";
    table.appendChild(tr);

    let num = i + 1;
    if(num == 10) num = 0;
    // ショートカット名
    let td1 = document.createElement('td');
    td1.width = "20%";
    tr.appendChild(td1);
    let b = document.createElement('b');
    b.textContent = "Alt + " + num.toString() + "　：　";
    td1.appendChild(b);

    // テキストボックス
    let td2 = document.createElement('td');
    td2.align = "left";
    td2.width = "80%";
    tr.appendChild(td2);
    let input = document.createElement('input')
    input.type = "text";
    input.id = "shortcut" + i.toString();
    input.maxlength = "100";
    input.size = "50";
    td2.appendChild(input);
  }

  // 保存ボタン - 初期化ボタン領域
  let div = document.createElement('div');
  div.height = "60";
  div.style = "height: 38px; margin-top: 10px;";
  body.appendChild(div);
  // 保存ボタン
  let buttonSave = document.createElement('button');
  buttonSave.className = "btn btn-sm btn-default";
  buttonSave.textContent = "　保存　";
  buttonSave.style = "float : left;";
  buttonSave.onclick = function() {
    for (let i=0; i<10; i++) 
    {
      let ele = document.getElementById("shortcut" + i.toString());
      if(ele == null) continue;

      setOpt(CHANNEL.name + "_shortcut" + i.toString(), ele.value);
    }

    modal.modal("hide");
    // if (getOpt(CHANNEL.name + "_small_emo") === "1") 
  }
  div.appendChild(buttonSave);

  function isNullShortcut() {
    for (let i=0; i<10; i++) 
    {
      let val = getOpt(CHANNEL.name + "_shortcut" + i.toString());
      if(val != null)
      {
        return false;
      }
    }
    return true;
  }
  function resetShortcut() {
    setOpt(CHANNEL.name + "_shortcut" + 0, "");
    setOpt(CHANNEL.name + "_shortcut" + 1, "");
    setOpt(CHANNEL.name + "_shortcut" + 2, "");
    setOpt(CHANNEL.name + "_shortcut" + 3, "");
    setOpt(CHANNEL.name + "_shortcut" + 4, "");
    setOpt(CHANNEL.name + "_shortcut" + 5, "");
    setOpt(CHANNEL.name + "_shortcut" + 6, "");
    setOpt(CHANNEL.name + "_shortcut" + 7, "");
    setOpt(CHANNEL.name + "_shortcut" + 8, "");
    setOpt(CHANNEL.name + "_shortcut" + 9, "");
  }

  function setInputShortcutOpt() {
    for (let i=0; i<10; i++) 
    {
      let ele = document.getElementById("shortcut" + i.toString());
      if(ele == null) continue;

      let val = getOpt(CHANNEL.name + "_shortcut" + i.toString());
      if(val != null)
      {
        ele.value = val;
      }
    }
  }
  setInputShortcutOpt();

  // ショートカットが存在しないなら初期化
  if(isNullShortcut() == true)
  {
    resetShortcut();
    setInputShortcutOpt();
  }

  // 初期化ボタン
  let buttonReset = document.createElement('button');
  buttonReset.className = "btn btn-sm btn-default";
  buttonReset.textContent = "　初期化　";
  buttonReset.style = "float : right;";
  buttonReset.onclick = function() {
    var result = window.confirm('ショートカットを初期化します。よろしいですか？');
    if(result == true)
    {
      resetShortcut();
      setInputShortcutOpt();
    }
  }
  div.appendChild(buttonReset);

  // フッター
  let fotter = document.createElement('div');
  fotter.className = "modal-fotter";
  inner2.appendChild(fotter);

  var modal = $("#shortcutlist")
  $('<button class="btn btn-sm btn-default" id="shortcutlistbtn" style="float:right;">ショートカット</button>').insertAfter("#commandlistbtn").on("click", function(event) {
    setInputShortcutOpt();
    // スイッチの切り替え
    modal.modal();
  });

  let isAltPress = false;
  let dc = document;//document.getElementById("chatline");//document;
  dc.addEventListener( "keydown" , function (event) {
    if(event.keyCode == 18) 
    {
      // デフォルト動作停止
      isAltPress = true;
    }
    //ここから
    if(isAltPress)
    {
      event.preventDefault();
    }
    //ここまで
  }, false);
  dc.addEventListener( "keyup" , function (event) {
    if(event.keyCode == 18)
    {
      isAltPress = false;
    }
  }, false);

  let chatline = document.getElementById("chatline");
  chatline.addEventListener( "keyup" , function (event) {
    if(isAltPress)
    {
      if(!(event.keyCode >= 48 && event.keyCode <= 57) && !(event.keyCode >= 96 && event.keyCode <= 105))
      {
        return;
      }

      let num = 0;
      if(event.keyCode >= 48 && event.keyCode <= 57)
      {
        num = event.keyCode - 48;
      }
      else if(event.keyCode >= 96 && event.keyCode <= 105)
      {
        num = event.keyCode - 96;
      }

      num -= 1;
      if(num == -1) num = 9;
      let val = getOpt(CHANNEL.name + "_shortcut" + num.toString());
      if(val != null)
      {
        this.value += val;
      }
    }
  }, false);
})();

// 再生中の動画へスクロールボタンの追加
(() => {
  $('<button class="btn btn-sm btn-default" id="scrollpl" size="1" title="再生中の動画へスクロール" style="float: right; height:20px; width:20px; padding: 0px;">▼</button>').insertBefore("#pllength");
  $("#scrollpl").on("click", function(event) {
    scrollQueue();
  });
})();

// チャット欄入力時コマンドの実装
(() => {
  // デフォルト処理を削除
  $("#chatline").off("keydown");

  // 再登録
  $("#chatline").keydown(function(ev) {
      // Enter/return
      if(ev.keyCode == 13) {
        if (CHATTHROTTLE) {
            return;
        }
        var msg = $("#chatline").val();
        msg = chatlineReplaceCommand(msg);
        if(msg.trim()) {
            var meta = {};
            if(msg.match(/\n/) !== null){ msg = "\n" + msg; }
            if (USEROPTS.adminhat && CLIENT.rank >= 255) {
                msg = "/a " + msg;
            } else if (USEROPTS.modhat && CLIENT.rank >= Rank.Moderator) {
                meta.modflair = CLIENT.rank;
            }

            // The /m command no longer exists, so emulate it clientside
            if (CLIENT.rank >= 2 && msg.indexOf("/m ") === 0) {
                meta.modflair = CLIENT.rank;
                msg = msg.substring(3);
            }

            socket.emit("chatMsg", {
                msg: msg,
                meta: meta
            });
            CHATHIST.push($("#chatline").val());
            CHATHISTIDX = CHATHIST.length;
            $("#chatline").val("");
        }
        return;
    }
    else if(ev.keyCode == 9) { // Tab completion
        try {
            chatTabComplete();
        } catch (error) {
            console.error(error);
        }
        ev.preventDefault();
        return false;
    }
    else if(ev.keyCode == 38) { // Up arrow (input history)
        if(CHATHISTIDX == CHATHIST.length) {
            CHATHIST.push($("#chatline").val());
        }
        if(CHATHISTIDX > 0) {
            CHATHISTIDX--;
            $("#chatline").val(CHATHIST[CHATHISTIDX]);
        }

        ev.preventDefault();
        return false;
    }
    else if(ev.keyCode == 40) { // Down arrow (input history)
        if(CHATHISTIDX < CHATHIST.length - 1) {
            CHATHISTIDX++;
            $("#chatline").val(CHATHIST[CHATHISTIDX]);
        }

        ev.preventDefault();
        return false;
    }
  });

  // メッセージコマンドの実装
  function chatlineReplaceCommand(msg){
    // 先頭置き換え系
    msg = commandOmikuji(msg);
    msg = commandDice(msg);

    // 文中置き換え系
    msg = commandShantae(msg);

    return msg;
  };

  // おみくじ
  function commandOmikuji(msg){
    let cmd = "!omikuti";
    if(msg.slice(0, 8) === cmd && (msg.length === 8 || msg.slice(9,10) === " "))
    {
      var result = "";

      var random = Math.floor( Math.random() * 61 );
      if(random >= 60) result = "順平";
      else if(random >= 50) result = "大吉";
      else if(random >= 40) result = "中吉";
      else if(random >= 30) result = "小吉";
      else if(random >= 20) result = "末吉"
      else if(random >= 10) result = "凶"
      else if(random >= 0) result = "大凶"

      msg = SYSMESSAGE + " " + "{0} {1} さんがおみくじを引きました → 【" + result + "】";
    }
    return msg;
  };

  // ダイス
  function commandDice(msg){
    let cmd = "!dice";
    if(msg.slice(0, 5) === cmd)
    {
      var ind = 6;
      var num1 = "";
      for (i = ind; i<msg.length; i++)
      {
        if(msg.slice(i,i+1) != " ")
        {
          num1 += msg.slice(i,i+1);
        }
        else
        {
          break;
        }
      }
      ind = i + 1;

      var num2 = "";
      for (i = ind; i<msg.length; i++)
      {
        if(msg.slice(i,i+1) != " ")
        {
          num2 += msg.slice(i,i+1);
        }
        else
        {
          break;
        }
      }

      var result = "";

      if(isNaN(num1) == false && isNaN(num2) == false)
      {
        if(num1 > num2)
        {
          var a = num1;
          num1 = num2;
          num2 = a;
        }

        var random = Math.floor( Math.random() * (num2 - num1 + 1) ) + Number(num1);
        msg = SYSMESSAGE + " " + "{0} {1} さんがサイコロを振りました(" + num1 + "~" + num2 + ") → 【" + random + "】";
      }
      else
      {
        msg = SYSMESSAGE + " " + "{0} {1} さんはサイコロを振れませんでした";
      }
    }
    return msg;
  };

  // ランダムシャンティ
  function commandShantae(msg){
    cmsg = " " + msg + " ";
    cmd = "!shantae";
    var ccmd = " " + cmd + " ";

    if ( cmsg.indexOf( ccmd ) != -1) {
      // 全部置き換えます
      while ( cmsg.indexOf( ccmd ) != -1) {
        var rep = "";
        var random = Math.floor( Math.random() * 9 );
        switch(random)
        {
          case 0: rep = "gbcshantaeshantae1"; break;
          case 1: rep = "gbcshantaeshantae2"; break;
          case 2: rep = "gbcshantaeshantae3"; break;
          case 3: rep = "gbcshantaeshantae4"; break;
          case 4: rep = "gbcshantaeshantae5"; break;
          case 5: rep = "gbcshantaeshantae6"; break;
          case 6: rep = "gbcshantaeshantae7"; break;
          case 7: rep = "gbcshantaeshantaekyougaku"; break;
          case 8: rep = "gbcshantaeshantaesyagami"; break;
        }

        cmsg = cmsg.replace(ccmd, " " + rep + " ");
      }
      return cmsg;
    }
    
    return msg;
  }

})();

// ツイートボタン
(() => {
  $('<div style="position: relative; float: left; margin: 5px;"></div><button class="btn btn-sm btn-default" style="padding: 0px 5px 0px 0px;" id="btn-tweet" title="ツイート"><img height="23" vspace="3" src="https://dl.dropboxusercontent.com/s/ncbstbqrc9gjdhi/tweetbutton.png"></button>').insertAfter('#voteskip');

  $("#btn-tweet").on("click", function(event) {
    openTwitter();
  });

  function openTwitter() {
    let text = "";
    if($("#channel_info").length) text = $("#channel_info").html();
    
    // 現在放送中の動画を記載
    text += "\r\n";
    let title = getPlayingTitle();
    if(title !== "")
    {
      text += "「" + title + "」を視聴中\r\n\r\n";
    }

    text = encodeURIComponent(text);

    let url = location.href;
    url = encodeURIComponent(url);
    let turl = "https://twitter.com/intent/tweet?text="+text+"&url="+url;
    window.open(turl, 'Twitter でリンクを共有する', 'width=552,height=480');
  }

  // 初回起動時の再生中URL
  function getPlayingTitle(){
    let lis = $("#queue").children('li');
    for(var i=0; i<lis.length; i++)
    {
      let li = lis.eq(i);
      if(li.hasClass('queue_active'))
      {
        return li.data("media").title;
      }
    }
    return "";
  }
})();