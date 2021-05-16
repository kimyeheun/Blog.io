/** ------------------------------------------------------------------------------------- /
 * ModuleName  : Service Core Module (v0.0.1)
 * Developer   : SeoulSystem.corp
 * DevLanguage : Javascript + jQuery
 * BuildDate   : 2018.01.26
 * Information : 연합뉴스 서비스 페이지에 필요한 UI 모듈을 정의한다.
 *               기본언어 Javascript + 개발기간 단축을 위해 jQuery 를 사용한다.
 * -------------------------------------------------------------------------------------- */

/** [JsSocials] Module Start ------------------------------------------------------ */
(function(window, $, undefined) {
    /** [jssocials Share] 는 Sns 공유를 처리하는 외부 라이브러리다. http://js-socials.com/ */
    var JSSOCIALS = "JSSocials",
        JSSOCIALS_DATA_KEY = JSSOCIALS;
    var getOrApply = function(value, context) {
        if($.isFunction(value)) {
            return value.apply(context, $.makeArray(arguments).slice(2));
        }
        return value;
    };
    var IMG_SRC_REGEX = /(\.(jpeg|png|gif|bmp|svg)$|^data:image\/(jpeg|png|gif|bmp|svg\+xml);base64)/i;
    var URL_PARAMS_REGEX = /(&?[a-zA-Z0-9]+=)?\{([a-zA-Z0-9]+)\}/g;
    var MEASURES = {
        "G": 1000000000,
        "M": 1000000,
        "K": 1000
    };
    var shares = {};
    function Socials(element, config) {
        var $element = $(element);
        $element.data(JSSOCIALS_DATA_KEY, this);
        this._$element = $element;

        this.shares = [];
        this._init(config);
        this._render();
    }
    Socials.prototype = {
        url: "",
        text: "",
        media : "",
        shareIn: "blank",

        showLabel: function(screenWidth) {
            return (this.showCount === false) ?
                (screenWidth > this.smallScreenWidth) :
                (screenWidth >= this.largeScreenWidth);
        },

        showCount: function(screenWidth) {
            return (screenWidth <= this.smallScreenWidth) ? "inside" : true;
        },

        smallScreenWidth   : 640,
        largeScreenWidth   : 1024,
        resizeTimeout      : 200,
        elementClass       : "jssocials",
        sharesClass        : "jssocials-shares",
        shareClass         : "jssocials-share",
        shareButtonClass   : "jssocials-share-button",
        shareLinkClass     : "jssocials-share-link",
        shareLogoClass     : "jssocials-share-logo",
        shareLabelClass    : "jssocials-share-label",
        shareLinkCountClass: "jssocials-share-linkcount",
        shareCountBoxClass : "jssocials-share-countbox",
        shareCountClass    : "jssocials-share-count",
        shareZeroCountClass: "jssocials-share-nocount",
        _init: function(config) {
            this._initDefaults();  //사이트 url 및 title 추출
            $.extend(this, config);
            this._initShares();
            this._attachWindowResizeCallback() ;
        },
        _initDefaults: function() {
            this.url = window.location.href;
            this.media = "";
            this.text = $.trim($("meta[name=description]").attr("content") || $("title").text());
        },
        _initShares: function() {
            this.shares = $.map(this.shares, $.proxy(function(shareConfig) {

                if(typeof shareConfig === "string") {
                    shareConfig = { share: shareConfig };
                }

                var share = (shareConfig.share && shares[shareConfig.share]);

                if(!share && !shareConfig.renderer) {
                    throw Error("Share '" + shareConfig.share + "' is not found");
                }

                return $.extend({
                    url: this.url,
                    text: this.text,
                    media : this.media
                }, share, shareConfig);
            }, this));
        },
        _attachWindowResizeCallback: function() {
            $(window).on("resize", $.proxy(this._windowResizeHandler, this));
        },
        _detachWindowResizeCallback: function() {
            $(window).off("resize", this._windowResizeHandler);
        },
        _windowResizeHandler: function() {
            if($.isFunction(this.showLabel) || $.isFunction(this.showCount)) {
                window.clearTimeout(this._resizeTimer);
                this._resizeTimer = setTimeout($.proxy(this.refresh, this), this.resizeTimeout);
            }
        },
        _render: function() {
            this._clear();

            this._defineOptionsByScreen();

            this._$element.addClass(this.elementClass);

            this._$shares = $("<div>").addClass(this.sharesClass)
                .appendTo(this._$element);

            this._renderShares();
        },
        _defineOptionsByScreen: function() {
            this._screenWidth = $(window).width();
            this._showLabel = getOrApply(this.showLabel, this, this._screenWidth);
            this._showCount = getOrApply(this.showCount, this, this._screenWidth);
        },
        _renderShares: function() {
            $.each(this.shares, $.proxy(function(_, share) {
                this._renderShare(share);
            }, this));
        },
        _renderShare: function(share) {
            var $share;
            if($.isFunction(share.renderer)) {
                $share = $(share.renderer());
            } else {
                $share = this._createShare(share);
            }

            $share.addClass(this.shareClass)
                .addClass(share.share ? "jssocials-share-" + share.share : "")
                .addClass(share.css)
                .appendTo(this._$shares);
        },
        _createShare: function(share) {
            var $result = $("<div>");
            var $shareLink = this._createShareLink(share).appendTo($result);
            if(this._showCount) {
                var isInsideCount = (this._showCount === "inside");
                var $countContainer = isInsideCount ? $shareLink : $("<div>").addClass(this.shareCountBoxClass).appendTo($result);
                $countContainer.addClass(isInsideCount ? this.shareLinkCountClass : this.shareCountBoxClass);
                this._renderShareCount(share, $countContainer);
            }
            return $result;
        },
        _createShareLink: function(share) {
            var shareStrategy = this._getShareStrategy(share);
            var $result = shareStrategy.call(share, {
                shareName : share.label,
                orgUrl    : share.url,
                shareUrl: this._getShareUrl(share)
            });

            $result.addClass(this.shareLinkClass)
                .append(this._createShareLogo(share));

            if(this._showLabel) {
                $result.append(this._createShareLabel(share));
            }

            $.each(this.on || {}, function(event, handler) {
                if($.isFunction(handler)) {
                    $result.on(event, $.proxy(handler, share));
                }
            });

            return $result;
        },
        _getShareStrategy: function(share) {
            var result = shareStrategies[share.shareIn || this.shareIn];
            if(!result)
                throw Error("Share strategy '" + this.shareIn + "' not found");
            return result;
        },
        _getShareUrl: function(share) {
            var shareUrl = getOrApply(share.shareUrl, share);
            return this._formatShareUrl(shareUrl, share);
        },
        _createShareLogo: function(share) {
            var logo = share.logo;

            var $result = IMG_SRC_REGEX.test(logo) ?
                $("<img>").attr("src", share.logo) :
                $("<i>").addClass(logo);

            $result.addClass(this.shareLogoClass);

            return $result;
        },
        _createShareLabel: function(share) {
            return $("<span>").addClass(this.shareLabelClass)
                .text(share.label);
        },
        _renderShareCount: function(share, $container) {
            var $count = $("<span>").addClass(this.shareCountClass);

            $container.addClass(this.shareZeroCountClass)
                .append($count);

            this._loadCount(share).done($.proxy(function(count) {
                if(count) {
                    $container.removeClass(this.shareZeroCountClass);
                    $count.text(count);
                }
            }, this));
        },
        _loadCount: function(share) {
            var deferred = $.Deferred();
            var countUrl = this._getCountUrl(share);

            if(!countUrl) {
                return deferred.resolve(0).promise();
            }

            var handleSuccess = $.proxy(function(response) {
                deferred.resolve(this._getCountValue(response, share));
            }, this);

            $.getJSON(countUrl).done(handleSuccess)
                .fail(function() {
                    $.get(countUrl).done(handleSuccess)
                        .fail(function() {
                            deferred.resolve(0);
                        });
                });

            return deferred.promise();
        },
        _getCountUrl: function(share) {
            var countUrl = getOrApply(share.countUrl, share);
            return this._formatShareUrl(countUrl, share);
        },
        _getCountValue: function(response, share) {
            var count = ($.isFunction(share.getCount) ? share.getCount(response) : response) || 0;
            return (typeof count === "string") ? count : this._formatNumber(count);
        },
        _formatNumber: function(number) {
            $.each(MEASURES, function(letter, value) {
                if(number >= value) {
                    number = parseFloat((number / value).toFixed(2)) + letter;
                    return false;
                }
            });

            return number;
        },
        _formatShareUrl: function(url, share) {
            return url.replace(URL_PARAMS_REGEX, function(match, key, field) {
                var value = share[field] || "";
                return value ? (key || "") + window.encodeURIComponent(value) : "";
            });
        },
        _clear: function() {
            window.clearTimeout(this._resizeTimer);
            this._$element.empty();
        },
        _passOptionToShares: function(key, value) {
            var shares = this.shares;
            $.each(["url", "text", "media"], function(_, optionName) {
                if(optionName !== key)
                    return;

                $.each(shares, function(_, share) {
                    share[key] = value;
                });
            });
        },
        _normalizeShare: function(share) {
            if($.isNumeric(share)) {
                return this.shares[share];
            }

            if(typeof share === "string") {
                return $.grep(this.shares, function(s) {
                    return s.share === share;
                })[0];
            }

            return share;
        },
        refresh: function() {
            this._render();
        },
        destroy: function() {
            this._clear();
            this._detachWindowResizeCallback();

            this._$element
                .removeClass(this.elementClass)
                .removeData(JSSOCIALS_DATA_KEY);
        },
        option: function(key, value) {
            if(arguments.length === 1) {
                return this[key];
            }

            this[key] = value;

            this._passOptionToShares(key, value);

            this.refresh();
        },
        shareOption: function(share, key, value) {
            share = this._normalizeShare(share);

            if(arguments.length === 2) {
                return share[key];
            }

            share[key] = value;
            this.refresh();
        }
    };
    $.fn.jsSocials = function(config) {
        var args = $.makeArray(arguments),
            methodArgs = args.slice(1),
            result = this;

        this.each(function() {
            var $element = $(this),
                instance = $element.data(JSSOCIALS_DATA_KEY),
                methodResult;

            if(instance) {
                if(typeof config === "string") {
                    methodResult = instance[config].apply(instance, methodArgs);
                    if(methodResult !== undefined && methodResult !== instance) {
                        result = methodResult;
                        return false;
                    }
                } else {
                    instance._detachWindowResizeCallback();
                    instance._init(config);
                    instance._render();
                }
            } else {
                new Socials($element, config);
            }
        });

        return result;
    };
    var setDefaults = function(config) {
        var component;

        if($.isPlainObject(config)) {
            component = Socials.prototype;
        } else {
            component = shares[config];
            config = arguments[1] || {};
        }

        $.extend(component, config);
    };
    var shareStrategies = {
        popup: function(args) {
        	/**
        	 * MEPS-14577
        	 * 중국어 위챗, 웨이보 공유 예외 추가
        	 * 위챗) pc웹 : qr코드, m웹 : copy+앱이동, 앱 : 네이티브 공유 기능 실행
        	 * 웨이보) pc웹 : 웨이보 공유서비스 이용, m웹 : copy+앱이동, 앱 : 네이티브 공유 기능 실행
        	 */
        	
        	var appChk = /YonhapnewsApp/i.test(navigator.userAgent);
        	try{ var appVersion = appChk && navigator.userAgent.match(/YonhapnewsApp-[\d|\.]*/) ? Number(navigator.userAgent.match(/YonhapnewsApp-[\d|\.]*/)[0].replace(/YonhapnewsApp|\.|-/g,'')) : 0;
        	}catch(e){ var appVersion = 0; }
        	var androidChk = /android/ig.test(navigator.userAgent);
        	var iosChk = /iphone|ipad|ipod/ig.test(navigator.userAgent);
        	
        	function openApp(scheme, marketURL, appStoreURL){	
        		var uagent = window.navigator.userAgent.toLowerCase();
        		if( /firefox|OPR/ig.test(uagent) ){
        			location.href = scheme;
        		}else if( /naver/ig.test(uagent) && androidChk ){
                	location.href = scheme;
        			var clickedAt = new Date;
        			var naverAppCheckTimer = setTimeout(function() {
        				if (new Date - clickedAt < 2000){
        					location.href = market;
        				}
        			}, 1500);
        		}else if( /ucbrowser|samsungbrowser|fxios|chrome|crios/ig.test(uagent) ){
        			location.href = scheme;
        			setTimeout(function(){
						if( !document.hidden ){
							location.href = iosChk ? appStoreURL : marketURL;
						}
					}, 2000);
        		}else if( /safari/ig.test(uagent) ){
        			var appPop = window.open(scheme, 'appOpen');
        			setTimeout(function(){
						if( document.hidden && (appPop == null || !appPop.closed) ){
							window.open(iosChk ? appStoreURL : marketURL, 'appOpen');
							appPop.close();
							location.href = iosChk ? appStoreURL : marketURL;
						}
					},4000);
        		}else{
        			location.href = scheme;
        			setTimeout(function(){
						if( !document.hidden ){
							location.href = iosChk ? appStoreURL : marketURL;
						}
					}, 2000);
        		}
            }
        	
            return $("<a>").attr("href","#none")
                .on("click", function(e) {
                	/* 위챗 */
                	if( args.shareName === 'Wechat' ){
                    	if( !appChk ){
                    		/* pc웹 */
                    		if( !location.href.match('m-cn') ){
                    			function getbarCode(url){
                                    return 'http://api.qrserver.com/v1/create-qr-code/?color=2dc100&data='+window.encodeURIComponent(url)
                                }
                                window.open(getbarCode(args.orgUrl), null, "width=250, height=250, location=0, menubar=0, resizeable=0, scrollbars=0, status=0, titlebar=0, toolbar=0");
                    		}
                    		/* m웹 */
                    		else{
                    			window.Clipboard ? (function(){
                    				var chk = Clipboard.copyConfirm(args.orgUrl, '链接已复制。允许打开APP？');
                    				if( chk ){
                    					openApp('weixin://', 'market://details?id=com.tencent.mm', 'https://itunes.apple.com/app/id414478124');
                    				}
                    			})() : '';
                    		}
                    	}
                    	/* 앱 */
                    	else {
                    		if( appVersion >= 101 ){
                    			if( androidChk ) { window.YonhapnewsApp ? window.YonhapnewsApp.appShare("com.tencent.mm", args.orgUrl) : Clipboard.copyAlert(args.orgUrl, '链接已复制。打开APP发送给好友。'); }
                    			else { location.href = 'wechat://'+encodeURIComponent(args.orgUrl); }
                    		}else{
                    			if( androidChk ){ window.YonhapnewsApp.shareUrlCopy(args.orgUrl); }
            					window.Clipboard ? Clipboard.copyAlert(args.orgUrl, '链接已复制。打开APP发送给好友。') : '';
                    		}
                    	}
                	}
                	/* 웨이보 */
                	else if( args.shareName === 'Weibo Sina' ){
                    	if( !appChk ) {
                    		/* pc웹 */
                    		if( !location.href.match('m-cn') ){
                    			window.open(args.shareUrl, null, "width=600, height=400, location=0, menubar=0, resizeable=0, scrollbars=0, status=0, titlebar=0, toolbar=0");
                    		}
                    		/* m웹 */
                    		else{
                    			window.Clipboard ? (function(){
                    				var chk = Clipboard.copyConfirm(args.orgUrl, '链接已复制。允许打开APP？');
                    				if( chk ){
                    					openApp('sinaweibo://gotohome', 'market://details?id=com.sina.weibo', 'https://itunes.apple.com/app/id350962117');
                    				}
                    			})() : '';
                    		}
                    	}
                    	/* 앱 */
                    	else{
                    		if( appVersion >= 101 ){
                    			if( androidChk ) { window.YonhapnewsApp ? window.YonhapnewsApp.appShare("com.sina.weibo", args.orgUrl) : Clipboard.copyAlert(args.orgUrl, '链接已复制。打开APP发送给好友。'); }
                    			else { location.href = 'sinaweibo://'+encodeURIComponent(args.orgUrl); }
                    		}else{
                    			if( androidChk ){ window.YonhapnewsApp.shareUrlCopy(args.orgUrl); }
            					window.Clipboard ? Clipboard.copyAlert(args.orgUrl, '链接已复制。打开APP发送给好友。') : '';
                    		}
                    	}
                	}
                	/* 위챗, 웨이보가 아닌 sns */
                	else{
                		window.open(args.shareUrl, null, "width=600, height=400, location=0, menubar=0, resizeable=0, scrollbars=0, status=0, titlebar=0, toolbar=0");
                	}
                	
                    return false;
                });
        },

        blank: function(args) {
            return $("<a>").attr({ target: "_blank", href: args.shareUrl });
        },

        self: function(args) {
            return $("<a>").attr({ target: "_self", href: args.shareUrl });
        }
    };
    window.jsSocials = {
        Socials: Socials,
        shares: shares,
        shareStrategies: shareStrategies,
        setDefaults: setDefaults
    };

}(window, jQuery));
(function(window, $, jsSocials) {
    $.extend(jsSocials.shares, {
        email: {
            label: "E-mail",
            logo: "fa fa-at",
            shareUrl: "mailto:{to}?subject={text}&body={url}",
            countUrl: "",
            shareIn: "self"
        },
        twitter: {
            label: "Tweet",
            logo: "fa fa-twitter",
            shareUrl: "https://twitter.com/share?url={url}&text={text}&via={via}&hashtags={hashtags}",
            countUrl: ""
        },
        facebook: {
            label: "Like",
            logo: "fa fa-facebook",
            shareUrl: "https://facebook.com/sharer/sharer.php?u={url}",
            countUrl: "https://graph.facebook.com/?id={url}",
            getCount: function(data) {
                return data.share && data.share.share_count || 0;
            }
        },
        vkontakte: {
            label: "Like",
            logo: "fa fa-vk",
            shareUrl: "https://vk.com/share.php?url={url}&title={text}&description={text}",
            countUrl: "https://vk.com/share.php?act=count&index=1&url={url}",
            getCount: function(data) {
                return parseInt(data.slice(15, -2).split(', ')[1]);
            }
        },
        googleplus: {
            label: "+1",
            logo: "fa fa-google",
            shareUrl: "https://plus.google.com/share?url={url}",
            countUrl: ""
        },
        linkedin: {
            label: "Share",
            logo: "fa fa-linkedin",
            shareUrl: "https://www.linkedin.com/shareArticle?mini=true&url={url}",
            countUrl: "https://www.linkedin.com/countserv/count/share?format=jsonp&url={url}&callback=?",
            getCount: function(data) {
                return data.count;
            }
        },
        pinterest: {
            label: "Pin it",
            logo: "fa fa-pinterest",
            shareUrl: "https://kr.pinterest.com/pin/create/bookmarklet/?media={media}&url={url}&description={text}",
            countUrl: "https://api.pinterest.com/v1/urls/count.json?&url={url}&callback=?",
            getCount: function(data) {
            	console.log(data);
                return data.count;
            }
        },
        stumbleupon: {
            label: "Share",
            logo: "fa fa-stumbleupon",
            shareUrl: "http://www.stumbleupon.com/submit?url={url}&title={text}",
            countUrl:  "https://cors-anywhere.herokuapp.com/https://www.stumbleupon.com/services/1.01/badge.getinfo?url={url}",
            getCount: function(data) {
                return data.result.views;
            }
        },
        telegram: {
            label: "Telegram",
            logo: "fa fa-paper-plane",
            shareUrl: "tg://msg?text={url} {text}",
            countUrl: "",
            shareIn: "self"
        },
        whatsapp: {
            label: "WhatsApp",
            logo: "fa fa-whatsapp",
            shareUrl: "whatsapp://send?text={url} {text}",
            countUrl: "",
            shareIn: "self"
        },
        line: {
            label: "LINE",
            logo: "fa fa-comment",
            shareUrl: "http://line.me/R/msg/text/?{text} {url}",
            countUrl: ""
        },
        viber: {
            label: "Viber",
            logo: "fa fa-volume-control-phone",
            shareUrl: "viber://forward?text={url} {text}",
            countUrl: "",
            shareIn: "self"
        },
        pocket: {
            label: "Pocket",
            logo: "fa fa-get-pocket",
            shareUrl: "https://getpocket.com/save?url={url}&title={text}",
            countUrl: ""
        },
        messenger: {
            label: "Share",
            logo: "fa fa-commenting",
            shareUrl: "fb-messenger://share?link={url}",
            countUrl: "",
            shareIn: "self"
        },
        redit: {
            label: "Redit",
            logo: "fa fa-redit",
            shareUrl: "http://www.reddit.com/submit?url={url}&title={text}",
            countUrl: ""
        },
        tumblr: {
            label: "Tumblr",
            logo: "fa fa-tumblr",
            shareUrl: "https://www.tumblr.com/widgets/share/tool?canonicalUrl={url}&title={text}&caption={text}",
            countUrl: ""
        },
        fbm: {
            label: "Fbm",
            logo: "fa fa-fbm",
            shareUrl: "http://www.facebook.com/dialog/send?app_id=1013291918759197&redirect_uri=https://www.yna.co.kr/program/fbm&link={url}%3Fsns=fbm&display=popup",
            countUrl: ""
        },
        /* 한국사이트 */
        naverBand : {
            label: "NaverBand",
            logo: "fa fa-naverBand",
            shareUrl: "http://www.band.us/plugin/share?body={text}&route={url}",
            countUrl: ""
        },
        naverBlog : {
            label: "NaverBlog",
            logo: "fa fa-naverBlog",
            shareUrl: "http://blog.naver.com/openapi/share?url={url}&title={text}",
            countUrl: ""
        },
        kakaoStory : {
            label: "KakaoStory",
            logo: "fa fa-kakaoStory",
            shareUrl: "https://story.kakao.com/share?url={url}",
            countUrl: ""
        },
        /* 중국사이트 */
        weibo : {
            label: "Weibo Sina",
            logo: "fa fa-weibo",
            shareUrl: "http://service.weibo.com/share/share.php?url={url}&appkey=&title={text}&pic=&ralateUid=&language=zh_cn",
            countUrl: ""
        },
        baidu : {
            label: "Baidu",
            logo: "fa fa-baidu",
            shareUrl: "http://tieba.baidu.com/i/app/open_share_api?link={url}",
            countUrl: ""
        },
        renren : {
            label: "Renren",
            logo: "fa fa-renren",
            shareUrl: "http://share.renren.com/share/buttonshare.do?link={url}&title={text}",
            countUrl: ""
        },
        douban : {
            label: "Douban",
            logo: "fa fa-douban",
            shareUrl: "https://www.douban.com/share/service?href={url}&image={img}",
            countUrl: ""
        },
        qzone : {
            label: "Qzone",
            logo: "fa fa-qzone",
            shareUrl: "http://sns.qzone.qq.com/cgi-bin/qzshare/cgi_qzshare_onekey?url={url}&title={text}",
            countUrl: ""
        },
        wechat : {
            label : "Wechat",
            logo: "fa fa-wechat",
            shareUrl: "",
            countUrl: ""
        },
        /* 일본사이트 */
        hatena : {
            label: "Hatena",
            logo: "fa fa-hatena",
            shareUrl: "http://b.hatena.ne.jp/entry/{url}",
            countUrl: ""
        },
        mixi : {
            label: "Mixi",
            logo: "fa fa-hatena",
            shareUrl: "http://mixi.jp/share.pl?u={url}",
            countUrl: ""
        },
        lineJp : {
            label: "line",
            logo: "fa fa-hatena",
            shareUrl: "http://line.me/R/msg/text/?{text}",
            countUrl: ""
        }
    });
}(window, jQuery, window.jsSocials));
/** [JsSocials] Module End -------------------------------------------------------- */

/** [YNA Core] Module Start ------------------------------------------------------------- */
(function(window, $){
    'use strict';
    var _YNA_Service;
    var _YNA_Style;
    var _YNA_PhotoView;
    var _YNA_ArticleView;
    var _YNA_GalleryView;
    var _YNA_TopNews;
    var _YNA_PhotoGrid;
    var _YNA_GeoLocation;
    var _YNA_Notification;
    var _YNA_Slider;
    var _YNA_AutoAlign;
    (function(){
        if (!Array.prototype.indexOf) {
            Array.prototype.indexOf = function(obj, start) {
                for (var i = (start || 0), j = this.length; i < j; i++) {if (this[i] === obj) {return i;}}
                return -1;
            };
        }
    })();

    /** 객체 정의 */
    window.YNA_CORE = null;
    window.YNA_SERVICE = {};

    /** Service Core module */
    _YNA_Service = function(props){
        this.state = {
            location : null,
            url : '',
            title : '',
            description : '',
            image : '',
            saved_page : false,
            saved_item_url : ''
        };
        this.control = {
            html : 'html',
            body : 'body',
            subContent : '.view-body',//'.sub-content',
            wrapContainer : '.wrap-container',
            mobileWrapContainer : '#wrapper',
            snsShareBox : '.article-view-zone .sns-share', //공유1
            snsShareBox2 : '.sns-flying', //공유2
            snsShareBox3 : '.pop-share .sns-share', //공유3
            snsShareBox4 : '.image-view-zone .sns-share', //공유4
            snsShareBox5 : '.social-btns .sns-share', //공유5(다국어 본문)
            savedShareBox : '.saved-list-zone ul',
            snsShareData : null, //공유모델
            btnLike : '.btn-like', //공감
            btnSave : '.btn-save', //즐겨찾기
            btnSave2 : '.btn-save02', //즐겨찾기2
            btnPrint : '.btn-print', //프린트
            fontSizeRadio : '.layer-font input[type="radio"]' //글자크기
        };
        this.props = {
            directoryName : 'view',
            articleKeyLength : 20,
            onSnsShare : null,
            onLike : null,
            onFavorite : null
        };
        this._init(props);
        return {
            'YNA' : '0.0.1',
            urlCopy : $.proxy(this._getUrlCopy, this),
            getArticleId : $.proxy(this._getArticleId, this),
            getUrlQuery : $.proxy(this._getUrlQueryParse, this),
            setNewUrlQuery : $.proxy(this._setNewUrlQuery, this),
            getArticleIdFromString : $.proxy(this._getArticleIdFromString, this),
            checkDevice : $.proxy(this._checkDevice, this),
            getUrlParam : $.proxy(this._getUrlParam, this),
            isViewPage : $.proxy(this._isViewPage, this),
            isSectionValue : $.proxy(this._isSectionValue, this),
            getUrlObject : $.proxy(this._getUrlObject, this)
        }
    };
    _YNA_Service.prototype = {
        _isSectionValue : function(){
            var urlQuery = this._getUrlQueryParse();
            var urlData = urlQuery['urlData'];
            var query = urlQuery['query'];
            var result;
            if(urlData.path.indexOf('/view/') !== -1){
                result = (query && query['section'])?query['section'].replace(/^\//gi,''):'';
            } else {
                var checkParam = urlData.path;
                var newParam = [];
                if( urlData.path || urlData.path === '' ){
                    checkParam = checkParam.split('/');
                    $.each(checkParam, function(num, val){
                        var isNumber = parseInt(val);
                        if( val !== '' && isNaN(isNumber) ){ newParam.push(val); }
                    });
                    result = newParam.join('/');
                } else {
                    result = 'index';
                }
            }
            return result;
        },
        _isViewPage : function(){
            var urlQuery = this._getUrlQueryParse();
            var urlData = urlQuery['urlData'];
            var query = urlQuery['query'];
            return decodeURIComponent(urlData.href).indexOf('/view/') !== -1;
        },
        _getUrlObject : function(newUrl){
            if( newUrl ){this._setNewUrlQuery(newUrl);}
            var query = this._getUrlQueryParse();
            var domain1 = null;
            var section = null;
            var cid = null;
            if( query.urlData && query.urlData.host ){
                domain1 = (query.urlData.host.split('.'))[0] || '';
                section = query.query['section'] || '';
                section = section.replace(/^\//gi,'');
                cid = this._getArticleId() || '';
            }
            return {
                domain1 : domain1,
                section : section,
                cid : cid
            }
        },
        _getUrlParam : function(){
            var thisSearch = window.location.search;
            var findParam = {};
            if( thisSearch ){
                var paramArray = thisSearch.replace('?','').split("&");
                for(var i=0; i<paramArray.length; i++){
                    var divGrp = paramArray[i].split('=');
                    findParam[divGrp[0]] = divGrp[1];
                }
            }
            return findParam;
        },
        _setState : function(){
            var linkLocation = window.location;
            this.state.location = {
                hash : linkLocation['hash'],
                host : linkLocation['host'],
                hostname : linkLocation['hostname'],
                href : linkLocation['href'],
                origin : linkLocation['origin'],
                search : linkLocation['search'],
                pathname : linkLocation['pathname'],
                protocol : linkLocation['protocol']
            };
            //this.state.location = window.location;
            this.control.html = $(this.control.html);
            this.control.body = $(this.control.body);
            var isWrapContainer = this.control.wrapContainer = $(this.control.wrapContainer);
            if( isWrapContainer.length === 0 ){
                this.control.wrapContainer = $(this.control.mobileWrapContainer);
            }
            this.control.subContent = $(this.control.subContent).children().first();
            this.control.snsShareBox = $(this.control.snsShareBox);
            this.control.snsShareBox2 = $(this.control.snsShareBox2);
            this.control.snsShareBox3 = $(this.control.snsShareBox3);
            this.control.snsShareBox4 = $(this.control.snsShareBox4);
            this.control.snsShareBox5 = $(this.control.snsShareBox5);
            this.control.savedShareBox = $(this.control.savedShareBox);
            this.control.btnPrint = $(this.control.btnPrint);
            this.control.btnLike = $(this.control.btnLike);
            this.control.btnSave = $(this.control.btnSave);
            this.control.btnSave2 = $(this.control.btnSave2);
            this.control.fontSizeRadio = $(this.control.fontSizeRadio);
        },
        _setMetaData : function(){
            var module  = this;
            var result = this.control.html.find('meta');
            $.each(result, function(){
                var item = $(this);
                var thisName = item.attr('name');
                var thisProps = item.attr('property');
                if( thisProps === 'og:url' || thisName === 'url' ){
                    module.state.url = item.attr('content');
                }
                if( thisProps === 'og:title' || thisName === 'title' ){
                    module.state.title = item.attr('content');
                }
                if( thisProps === 'og:description' || thisName === 'description' ){
                    module.state.description = item.attr('content');
                }
                if( thisProps === 'og:image' || thisName === 'image' ){
                    module.state.image = item.attr('content');
                }
            });
        },
        _setShareBox : function(){
            var snsDataBody = $('.shareDataBox');
            var snsBody = $('<div class="shareDataBox"></div>');
            if( snsDataBody.length === 0 ){
                this.control.wrapContainer.append(snsBody.hide());
                this.control.snsShareData = $('.shareDataBox');
            } else {
                this.control.snsShareData = snsDataBody;
            }
        },
        _setShareModule : function(module, share){
            var module = module || this;
            var shareDataBox = $(".shareDataBox");
            if( shareDataBox.length > 0 ){
                /** TODO : 공유 대상이 늘어나면 이곳을 수정해야함 */
                var sharesOjs = {
                    url: module.state.url,
                    text: module.state.description,
                    media : module.state.image,
                    shareIn: "popup",
                    showLabel: true, //"inside"
                    showCount: false,
                    shares: [
                        { share : "facebook", label : "Facebook"},
                        { share : "twitter", label : "Twitter"},
                        { share : "googleplus", label : "Google+"},
                        { share : "line", label : "LINE"},
                        { share : "pinterest", label : "Pinterest"},
                        { share : "linkedin", label : "LinkedIn"},
                        { share : "redit", label : "Redit"},
                        { share : "tumblr", label : "Tumblr"},
                        { share : "fbm", label : "Fbm"},
                        { share : "wechat", label : "Wechat"},
                        { share : "weibo", label : "Weibo Sina"},
                        { share : "baidu", label : "Baidu"},
                        { share : "douban", label : "Douban "},
                        { share : "qzone", label : "Qzone "},
                        { share : "renren", label : "Renren "},
                        { share : "hatena", label : "Hatena "}
                    ]
                };
                if( share ){
                    sharesOjs['url'] = share['url'];
                    sharesOjs['text'] = share['text'];
                    sharesOjs['media'] = share['media'];
                }
                shareDataBox.jsSocials("destroy");
                shareDataBox.jsSocials(sharesOjs);
            }
        },
        _savedItemShare : function(item){
            var parentItem = item.parents('li');
            if( parentItem.length === 1 ){
                var url = parentItem.attr('data-url') || '';
                var text = parentItem.find('.tit').text() || '';
                var media = parentItem.find('img').attr('src') || '';
                this.state.saved_page = true;
                this.state.saved_item_url = url;
                this._setShareModule(this, {
                    url : url,
                    text : text,
                    media : media
                });
            }
        },
        _setSnsShareFnc : function(node){
            //var httpLink = window.location.protocol;
            var httpLink = 'http:';
            var item = $(node);
            var thisSnsLink = item.attr('data-snslink');
            var thisSnsTitle = item.attr('data-title');
            var thisSnsthumbnail = $("meta[property = 'og:image']").attr("content");
            if( thisSnsLink && thisSnsLink !== '' ){
                // 모바일은 m-global/js/common.js 에서 따로 처리함
                if( location.href.match('cb.yna.co.kr/gate/big5/cn') ) {
                    var thisSnsLink = thisSnsLink.replace(/(.*[yna\.kr|view]\/)(.*)(\?)*/,'$2');
                    thisSnsLink = 'https://cb.yna.co.kr/gate/big5/cn.yna.co.kr/view/'+thisSnsLink;
                }
                if( thisSnsLink.indexOf('http') < 0 ){ thisSnsLink = httpLink+thisSnsLink }
                this._setShareModule(this, {url : thisSnsLink, text : thisSnsTitle, media : thisSnsthumbnail});
            }
            this._onShareHandler(node);
        },
        _setNewUrlQuery : function(url){
            if( url ){
                var totalUrl = url.split('//');
                var href = totalUrl[0];
                var host = totalUrl[1];
                var search = '?'+(url.split("?"))[1];
                var hostDiv = host.split('/');
                var origin = href+'//'+host.substr(0, hostDiv[0].length);
                var pathname = url.substr(origin.length, url.length);
                pathname = pathname.substr(0, pathname.length-search.length);
                this.state.location = {
                    hash : '',
                    host : hostDiv[0],
                    hostname : hostDiv[0],
                    href : url,
                    origin : origin,
                    search : search,
                    pathname : pathname,
                    protocol : href
                };
            }
        },
        _getUrlQueryParse : function(){
            /** Url Query Parse Fnc */
            var location = this.state.location;
            var search = location.search;
            var query;
            var queryObject;
            try {
                queryObject = {
                    urlData : {
                        origin : location.origin ? location.origin : location.protocol + "//" + location.hostname + (location.port ? ':' + location.port: ''),
                        hash : location.hash,
                        href : location.href,
                        host : location.hostname,
                        path : location.pathname,
                        search : location.search
                    },
                    query : {}
                };
                if(search || search !== '' ){
                    query = (search.split('?')[1]).split('&');
                    $.each( query, function(n, v){
                        var value = v.split('=');
                        if( value[0] ){queryObject['query'][value[0]] = value[1] || '';}
                    });
                }
                return queryObject;
            } finally {
                location = null;
                search = null;
                query = null;
                queryObject = null;
            }
        },
        _getUrlCopy : function(copyLink){
            var newCopyInput = $('<input>').attr('id','urlLinkCoupInput');
            var copyUrl = window.location.href;
            if( copyLink != null && copyLink != undefined ) {
                copyUrl = copyLink;
                if( copyLink.indexOf('cb') >= 0 ) {
                    copyUrl = 'https://cb.yna.co.kr/gate/big5/cn.yna.co.kr/view/' + copyLink.replace(/(.*[yna\.kr|view]\/)(.*)/,'$2');
                }
            }
            var sitePathName = window.location.pathname;
            if( this.state.saved_page ){ copyUrl = this.state.saved_item_url || ''; }
            if(!sitePathName.match("/saved/index")) {
            	$('body').append(newCopyInput);
                (newCopyInput.val(copyUrl))[0].select();
                document.execCommand("Copy");
                newCopyInput.remove();
            }
        },
        _getArticleId : function(){
            var path = this.state.location.pathname;
            var splitString = '/'+(this.props.directoryName || 'view')+'/';
            var articleId;
            var idLength;
            var realId;
            try {
                articleId = path.split(splitString)[1];
                idLength = this.props.articleKeyLength || 20;
                realId = (articleId)?articleId.substr(0, idLength):null;
                if( realId ){
                    return realId;
                } else {
                    return null;
                }
            } catch (err){
                console.log('fail! load article id');
            }finally {
                path = null;
                splitString = null;
                articleId = null;
                idLength = null;
                realId = null;
            }
        },
        _getArticleIdFromString : function(string){
            var result;
            if( string && string.indexOf('/view/') !== -1 ){
                result = string.split('/view/');
                result = result[result.length-1];
                result = result.substr(0, 20);
            }else{
                result = string.split('/yna.kr/');
                result = result[result.length-1];
                result = result.substr(0, 20);
            }
            var cidReg = /[A-Z]{3}\d{17}/;
            if( !cidReg.test(result) )
                result = 'NotValid';
            return result;
        },
        _onPrintHandler : function(){
            if( location.href.match('www.yna') ) return;
            var title = $('h1.tit');
            var UI = window.YNA_SERVICE['YNA_UI'];
            var urlData = UI.getUrlQuery()['urlData'];
            try {
                var cattr = UI.getArticleId().slice(0,1);
            } catch(e) { }
            var printUrl = '';
            if( cattr == 'A' ) printUrl = urlData['origin'] + urlData['path'] + '?section=print';
            else if( cattr == 'P' ) printUrl = urlData['origin'] + urlData['path'] + '?section=photo-print';
            else if( cattr == 'G' ) printUrl = urlData['origin'] + urlData['path'] + '?section=graphic-print';
            else if( cattr == 'M' ) printUrl = urlData['origin'] + urlData['path'] + '?section=video-print';
            else printUrl = urlData['href'];
            // status=1,toolbar=1,height=600,width=600,menubar=1
            var printPopup = window.open(printUrl,"_print","left=0, top=0, width=750, height=800, scrollbars=yes, resizable=1");
            printPopup.print();
        },
        _onFavoriteHandler : function(node, type){
            if( typeof this.props.onFavorite === 'function' ){
                if( type == 'save1' ) var item = $(node).parents('.btn-save');
                else var item = $(node).parents('.btn-save02');
                var saveid = $('#cid').val() || '';
                var isFavorite = (item.hasClass('on'))?'off':'on';
                this.props.onFavorite(isFavorite, saveid);
            }
        },
        _onLikeHandler : function(node){
            if( typeof this.props.onLike === 'function' ){
                var item = $(node).parents('.btn-like');
                var likeId = item.attr('data-likeid') || '';
                var isLike = (item.hasClass('btnlike-on'))?'off':'on';
                this.props.onLike(isLike, likeId);
            }
        },
        _onShareHandler : function(node){
            var item = $(node);
            var type = item[0].classList[1];
            var triggerTarget = '';
            var shareString = 'jssocials-share-';
            /** TODO : 공유 대상이 늘어나면 이곳을 수정해야함 */
            switch(type){
                case 'fb'   : triggerTarget = 'facebook'; break;
                case 'tw'   : triggerTarget = 'twitter'; break;
                case 'ggp'  : triggerTarget = 'googleplus'; break;
                case 'pin'  : triggerTarget = 'pinterest'; break;
                case 'lin'  : triggerTarget = 'linkedin'; break;
                case 'tum'  : triggerTarget = 'tumblr'; break;
                case 'red'  : triggerTarget = 'redit'; break;
                case 'fbm'  : triggerTarget = 'fbm'; break;
                case 'wei'  : triggerTarget = 'weibo'; break;
                case 'wec'  : triggerTarget = 'wechat'; break;
                case 'qq'   : triggerTarget = 'qzone'; break;
                case 'ren'  : triggerTarget = 'renren'; break;
                case 'hb'  : triggerTarget = 'hatena'; break;
                case 'line'  : triggerTarget = 'line'; break;
                case 'copy' :
                    shareString = null;
                    triggerTarget = null;
                    break;
            }
            shareString = shareString+triggerTarget;
            if( shareString ){
                var snsTarget = this.control.snsShareData.find('.'+shareString+' a');
                snsTarget.trigger('click');

                if( typeof this.props.onSnsShare === 'function' ){
                    /** TODO : 요청에 의해 버튼에 sns 링크 값에서 id 값을 추출해서 인자로 넘기는걸로 수정.
                     * 하지만... view 가 아닐 수 있으므로 수정이 필요함.
                     * */
                    var snsId = item.attr('data-snslink');
                    snsId = this._getArticleIdFromString(snsId);
                    if( snsId == 'NotValid' ) return;
                    this.props.onSnsShare(snsId);
                }
            } else {
                var copyLink = item.attr('data-snslink');
                this._getUrlCopy(copyLink);
            }
        },
        _setControl : function(){
            var module = this;
            var body = $('body');
            var control = module.control;
            var PhotoBox = $(".photo-grid");
            /** 기사 좋아요 클릭 이벤트 */
            body.on('click', '.btn-like', function(e){
                module._onLikeHandler(e.target);
                return false;
            });
            /** 기사 즐겨찾기 등록 이벤트 */
            body.on('click', '.btn-save', function(e){
                module._onFavoriteHandler(e.target, 'save1');
                return false;
            });
            body.on('click', '.btn-save02', function(e){
                module._onFavoriteHandler(e.target, 'save2');
                return false;
            });
            /** 프린트 버튼 이벤트 */
            control.btnPrint.off('click.btn_print');
            control.btnPrint.on('click.btn_print', function(){
                module._onPrintHandler();
                return false;
            });
            /** 폰트사이즈 증감시 추가 - lineHeight 을 수정한다. */
            control.fontSizeRadio.on('change', function(){
                if( control.subContent.css('line-height') !== 'initial' ){
                    control.subContent.css('line-height','initial');
                }
            });
            /** 공유 버튼 이벤트 */
            body.off('click.sns_side', '.sns-flying button.share-btn02:not(.mor), .sns-flying button.share-btn:not(.mor)');
            body.on('click.sns_side', '.sns-flying button.share-btn02:not(.mor), .sns-flying button.share-btn:not(.mor)', function(){
                module._setSnsShareFnc(this);
                return false;
            });
            body.off('click.sns_share', '.pop-share .sns-share button.share-btn');
            body.on('click.sns_share', '.pop-share .sns-share button.share-btn', function(){
                module._setSnsShareFnc(this);
                return false;
            });
            body.off('click.sns_share', '.social-btns .sns-share button.share-btn');
            body.on('click.sns_share', '.social-btns .sns-share button.share-btn', function(){
                module._setSnsShareFnc(this);
                return false;
            });
            control.savedShareBox.off().on('click', 'li .sns-share button.share-btn', function(e){
                module._setSnsShareFnc(this);
                e.preventDefault();
                return false;
            });
            if( PhotoBox && PhotoBox.length > 0 ){
                PhotoBox.on('click', '.sns-share button.share-btn', function(){
                    module._setSnsShareFnc(this);
                    return false;
                });
            }

            /** 모바일용 공통 공유 박스의 이벤트 처리 */
            var mobileSnsBtn = $('#sharePop .pop-share .btn-group');
            mobileSnsBtn.off().on('click.sns_share', 'button.share-btn', function(e){
                module._setSnsShareFnc(this);
                e.preventDefault();
                return false;
            });

            var mobileSnsBtn02 = $('.social-btns .social-right');
            mobileSnsBtn02.off().on('click', '.share-btn', function(e){
                module._setSnsShareFnc(this);
                e.preventDefault();
                return false;
            });
        },
        _checkDevice : function(){
            var currentDevice, currentOS, yonhapDevice = 'N';
            currentDevice = (/iphone|ipod|android/i.test(navigator.userAgent.toLowerCase()));
            if (currentDevice) {
                var userAgent = navigator.userAgent.toLowerCase();
                if (userAgent.search("android") > -1){
                    currentOS = "android";
                }else if ((userAgent.search("iphone") > -1) || (userAgent.search("ipod") > -1) || (userAgent.search("ipad") > -1)) {
                    currentOS = "ios";
                } else {
                    currentOS = "else";
                }
                currentDevice = "mobile";
            } else {
                currentOS = "desktop";
                currentDevice = "desktop";
            }
            if(/YonhapnewsApp/i.test(navigator.userAgent)){
                yonhapDevice = 'Y';
                currentDevice = "mobile";
            }
            return {
                'currentDevice' : currentDevice,
                'currentOS' : currentOS,
                'yonhapDevice' : yonhapDevice
            };
        },
        _init : function(props){
            $.extend(this.props, props);
            this._setState();
            this._setMetaData();
            this._setShareBox();
            this._setShareModule();
            this._setControl();
        }
    };
    _YNA_Service.prototype.constructor = _YNA_Service;

    /** Style module */
    _YNA_Style = function(props){
        this.props = {
            fxLoadBar : false,
            fxLoadBarCol : '#00bcd4'
        };
        this.state = {};
        this.dom = {header : '#header'};
        this.init(props);
    };
    _YNA_Style.prototype = {
        setActiveDom : function(module){
            $.each(module.dom, function(key, val){
                module.dom[key] = $(val);
            });
        },
        setHeaderLoadBar : function(){
            var header = this.dom.header;
            if( $('.yna-fx-loader').length === 0 ){
                var newLoader = $('<div class="yna-fx-loader"></div>');
                newLoader.css({
                    'position' : 'absolute',
                    'display' : 'none',
                    // 'z-index' : 9998,
                    'height' : '3px',
                    // 'bottom' : '-3px',
                    'top' : '132px',
                    'width'  : '0%',
                    'background-color' : this.props.fxLoadBarCol,
                    'transition': 'width .4s ease-out',
                    'transform': 'rotateZ(0deg)'
                });
                header.append(newLoader);
            }
        },
        setHeaderLoadStyle : function(target, evt){
            var loadBar = $('.yna-fx-loader');
            var scrollTop = parseInt($(target).scrollTop());
            if( $('.urgent-news').css('display') != 'none' ) var urgentNewsH = $('.urgent-news').height();
            else var urgentNewsH = 0;
            var headerH = 210; //this.dom['header'].height() || 0;
            var contentH = $('.wrap-container').height() || 0;
            var footH = $('#footer').height() | 0;
            var totalH = urgentNewsH+headerH+contentH+footH;
            var realScroll = scrollTop + $(window).height();
            var barWidth = parseInt((realScroll/totalH)*100);
            if( this.dom['header'].hasClass('smenu-on') ){
                if(barWidth > 100){ barWidth = 100;}
                loadBar.css('width', barWidth+'%').show();
            }else{
                loadBar.css('width', 0).hide();
            }
        },
        loadModuleCheck : function(){
            if( this.props.fxLoadBar ){
                this.setHeaderLoadBar();
            }
        },
        control : function(){
            var dom = this.dom;
            var module = this;
            $(window).on('scroll', function(evt){
                module.setHeaderLoadStyle(this, evt);
            });
        },
        init : function(props){
            $.extend(this.props, props);
            this.setActiveDom(this);
            this.loadModuleCheck();
            this.control();
        }
    };
    _YNA_Style.prototype.constructor = _YNA_Style;

    /** photo view left/right move module */
    _YNA_PhotoView = function(props){
        this.props = props;
        this.dom = {
            btnPrev : '.btn-arrow.prev',
            btnNext : '.btn-arrow.next',
            mBtnPrev : '.btn-prev button',
            mBtnNext : '.btn-next button'
        };
        this.state = {
            cid : null,
            cidFirst : '',
            langType : null,
            section : '',
            deviceType : null
        };

        this.defaultImgPath = {
            'EN' : 'image/general',
            'CK' : 'image/photos',
            'JP' : 'image/photos',
            'AR' : 'image/photos',
            'SP' : 'image/photos',
            'FR' : 'image/photos'
        };
        this.init(this);
    };
    _YNA_PhotoView.prototype = {
        activeDom : function(){
            this.dom.btnPrev = (this.props.btnPrev)?$(this.props.btnPrev):$(this.dom.btnPrev);
            this.dom.btnNext = (this.props.btnNext)?$(this.props.btnNext):$(this.dom.btnNext);
            this.dom.mBtnPrev = (this.props.mBtnPrev)?$(this.props.mBtnPrev):$(this.dom.mBtnPrev);
            this.dom.mBtnNext = (this.props.mBtnNext)?$(this.props.mBtnNext):$(this.dom.mBtnNext);
            this.state.query = this.props.UI.getUrlQuery();
            this.state.cid = this.props.UI.getArticleId();
            if( !this.state.cid ){ return false; }
            this.state.deviceType = this.props.UI.checkDevice();
            this.state.section = this.props.UI.isSectionValue();
            if( this.state.deviceType['currentDevice'] != 'mobile' ){
                if( this.state.section.length < 1 || this.state.section == 'news' ) this.state.section = this.defaultImgPath[LANG_TYPE.toUpperCase()];
            }
            var cidFirst = this.state.cidFirst = this.state.cid.substr(0, 1);
            return cidFirst === 'P' || cidFirst === 'G';
        },
        onRequestLangType : function(){
            if( typeof this.props.onRequestLangType === 'function' ){
                if( this.state.cid ){
                    this.props.onRequestLangType(this, this.state.query);
                    this.onRequest();
                }
            }
        },
        setState : function(name, value){
            this.state[name] = value;
        },
        onRequest : function(){
            if( typeof this.props.request === 'function' ){
                if( this.state.cid ){
                    this.props.request(this, this.state.langType, this.state.cid);
                }
            }
        },
        onRender : function(param){
            if(typeof this.props.render === 'function' ){
                this.props.render({
                    dom : this.dom,
                    query : this.state.query,
                    "default" : this.defaultImgPath,
                    langType : this.state.langType,
                    data : param
                });
            }
        },
        setPrevNextItem : function(param){
            var result;
            if( param.length > 0 ){
                var prev = param[0];
                var next = param[1] || param[0];
                var cid = this.state.cid;
                $.each(param, function(num, val){
                    var thisCid = val['CID'];
                    if( cid === thisCid ){
                        var prevNum = num-1;
                        var nextNum = num+1;
                        if( num === 0 ){prevNum = param.length-1;}
                        if( num === param.length-1 ){nextNum = 0;}
                        prev = param[prevNum];
                        next = param[nextNum];
                        return false;
                    }
                });
                result = {prev : prev, next : next};
            }
            this.onRender(result)
        },
        init : function(){
            if( location.host.match('m.yna')|| location.host.match('www') || location.host.match('m-') ) return;
            if( this.activeDom() ){
                this.onRequestLangType();
            }
        }
    };
    _YNA_PhotoView.prototype.constructor = _YNA_PhotoView;

    /** Article view left/right move module */
    _YNA_ArticleView = function(props){
        this.props = props;
        this.dom = {
            btnPrev : '.btn-a-prev', //이전기사
            btnNext : '.btn-a-next' //다음기사
        };
        this.state = {
            cid : null,
            langType : null
        };
        this.init(this);
    };
    _YNA_ArticleView.prototype = {
        activeDom : function(){
            this.dom.btnPrev = (this.props.btnPrev)?$(this.props.btnPrev):$(this.dom.btnPrev);
            this.dom.btnNext = (this.props.btnNext)?$(this.props.btnNext):$(this.dom.btnNext);
            this.state.query = this.props.UI.getUrlQuery();
            this.state.cid = this.props.UI.getArticleId();
            if( !this.state.cid ){ return false; }
            return this.state.cid.substr(0, 1) === 'A';
        },
        onRequestLangType : function(){
            if( typeof this.props.onRequestLangType === 'function' ){
                if( this.state.cid ){
                    this.props.onRequestLangType(this, this.state.query);
                    this.onRequest();
                }
            }
        },
        setState : function(name, value){
            this.state[name] = value;
        },
        setPrevNextItem : function(param){
            var result;
            if( param.length > 0 ){
                var prev = param[0];
                var next = param[1] || param[0];
                var cid = this.state.cid;
                $.each(param, function(num, val){
                    var thisCid = val['CID'];
                    if( cid === thisCid ){
                        var prevNum = num-1;
                        var nextNum = num+1;
                        if( num === 0 ){prevNum = param.length-1;}
                        if( num === param.length-1 ){nextNum = 0;}
                        prev = param[prevNum];
                        next = param[nextNum];
                        return false;
                    }
                });
                result = {prev : prev, next : next};
            }
            this.onRender(result)
        },
        onRequest : function(){
            if( typeof this.props.request === 'function' ){
                if( this.state.cid ){
                    this.props.request(this, this.state.langType, this.state.cid);
                }
            }
        },
        onRender : function(param){
            if(typeof this.props.render === 'function' ){
                this.props.render({
                    dom : this.dom,
                    query : this.state.query,
                    data : param
                });
            }
        },
        init : function(){
            if( location.host.match('m.yna') || location.host.match('www') || location.host.match('m-') ) return;
            if( this.activeDom() ){
                this.onRequestLangType();
            }
        }
    };
    _YNA_ArticleView.prototype.constructor = _YNA_ArticleView;

    /** Gallery view left/right move module */
    _YNA_GalleryView = function(props){
        this.props = props;
        this.dom = {
            btnPrev : '.btn-arrow.prev', //이전기사
            btnNext : 'btn-arrow.next' //다음기사
        };
        this.state = {
            cid : null,
            langType : null
        };
        this.init(this);
    };
    _YNA_GalleryView.prototype = {
        activeDom : function(){
            this.dom.btnPrev = (this.props.btnPrev)?$(this.props.btnPrev):$(this.dom.btnPrev);
            this.dom.btnNext = (this.props.btnNext)?$(this.props.btnNext):$(this.dom.btnNext);
            this.state.query = this.props.UI.getUrlQuery();
            this.state.cid = this.props.UI.getArticleId();
            if( !this.state.cid ){ return false; }
            return this.state.cid.substr(0, 3) === 'IPT';
        },
        onRequestLangType : function(){
            if( typeof this.props.onRequestLangType === 'function' ){
                if( this.state.cid ){
                    this.props.onRequestLangType(this, this.state.query);
                    this.onRequest();
                }
            }
        },
        setState : function(name, value){
            this.state[name] = value;
        },
        setPrevNextItem : function(param){
            var result;
            if( param.length > 0 ){
                var prev = param[0];
                var next = param[1] || param[0];
                var cid = this.state.cid;
                $.each(param, function(num, val){
                    var thisCid = val['CID'];
                    if( cid === thisCid ){
                        var prevNum = num-1;
                        var nextNum = num+1;
                        if( num === 0 ){prevNum = param.length-1;}
                        if( num === param.length-1 ){nextNum = 0;}
                        prev = param[prevNum];
                        next = param[nextNum];
                        return false;
                    }
                });
                result = {prev : prev, next : next};
            }
            this.onRender(result);
        },
        onRequest : function(){
            if( typeof this.props.request === 'function' ){
                if( this.state.cid ){
                    this.props.request(this, this.state.langType, this.state.cid);
                }
            }
        },
        onRender : function(param){
            if(typeof this.props.render === 'function' ){
                this.props.render({
                    dom : this.dom,
                    query : this.state.query,
                    data : param
                });
            }
        },
        init : function(){
            if( location.host.match('m.yna') || location.host.match('www') || location.host.match('m-') ) return;
            if( this.activeDom() ){
                this.onRequestLangType();
            }
        }
    };
    _YNA_GalleryView.prototype.constructor = _YNA_GalleryView;

    /** Date List Module */
    _YNA_TopNews = function(){
        this.props = null;
        this.state = {
            todayDate : null,
            dateType  : 'week',
            sDate     : null,
            eDate     : null,
            isActive  : false,
            page	  : 1,
            count	  : 15,
            total	  : 0,
            wSDate	  : null,
            wEDate	  : null
        };
        this.dom = {
            dateChangeControl : null,
            weekMonthControl : null,
            dateListDom : null
        };
        this.snsInfo = {url: null, text: null, media: null};
        return {
            init : $.proxy(this.init, this),
            getDate : $.proxy(this.getDate, this)
        }
    };
    _YNA_TopNews.prototype = {
        setDate : function(dateType, sDate, eDate, wSDate, wEDate){
            this.state.dateType = dateType;
            this.state.sDate = sDate;
            this.state.eDate = eDate;
            this.state.wSDate = wSDate;
            this.state.wEDate = wEDate;
        },
        getDate : function(d){
            var date = (d)?new Date(d):new Date();
            var year = parseInt(date.getFullYear());
            var month = parseInt(date.getMonth());
            var monthNum = month+1;
            var monthStr = ('0'+monthNum);
            monthStr = monthStr.substr(monthStr.length-2, 2);
            var day = parseInt(date.getDate());
            var dayStr = ('0'+day);
            dayStr = dayStr.substr(dayStr.length-2, 2);
            var fullString = year+monthStr+dayStr;
            return {
                year : year,
                monthNum : monthNum,
                monthStr : monthStr,
                day : day,
                dayStr : dayStr,
                fullString : fullString,
                date : date
            }
        },
        onCheckDateListBody : function(){
            var findDom = $('body').find(this.props.className);
            if( findDom && findDom.length === 1 ){
                this.dom.dateChangeControl = findDom.find(this.props.dateChangeGroup);
                this.dom.weekMonthControl = findDom.find(this.props.weekMonthGroup);
                this.dom.dateListDom = findDom.find(this.props.dateListGroup);
                return true;
            } else {
                return false;
            }
        },
        getBeforeDayDate : function(num, today){
            var d = (today)?new Date(today):new Date();
            var dayOfMonth = d.getDate();
            var monthOfYear = d.getMonth();
            d.setDate(dayOfMonth + num);
            return this.getDate(d);
        },

        setMakeDateChangeDom : function(){
            var sDate = this.state.sDate;
            var eDate = this.state.eDate;
            var today = this.state.todayDate;
            var lang = LANG_TYPE;
            var dateType = this.state.dateType;

            if( lang == 'en' ){
                var sDateString = sDate.date.Format2('MMM',lang)+'.'+sDate.dayStr;
                var eDateString = eDate.date.Format2('MMM',lang)+'.'+eDate.dayStr;
            }else if( lang == 'ck' ){
                var sDateString = sDate.date.Format2('MMMM',lang)+sDate.dayStr+'日';
                var eDateString = eDate.date.Format2('MMMM',lang)+eDate.dayStr+'日';
            }else if( lang == 'jp' ){
                var sDateString = sDate.monthStr+'.'+sDate.dayStr;
                var eDateString = eDate.monthStr+'.'+eDate.dayStr;
            }else if( lang == 'ar' ){
                var sDateString = sDate.monthStr+'.'+sDate.dayStr;
                var eDateString = eDate.monthStr+'.'+eDate.dayStr;
            }else if( lang == 'sp' ){
                var sDateString = sDate.day+' de '+sDate.date.Format2('MMM',lang).toLowerCase()+' ';
                var eDateString = eDate.day+' de '+eDate.date.Format2('MMM',lang).toLowerCase()+'';
            }else if( lang == 'fr' ){
                var sDateString = sDate.dayStr+'.'+sDate.monthStr;
                var eDateString = eDate.dayStr+'.'+eDate.monthStr;
            }

            var sDateNumString = sDate.year+'-'+sDate.monthStr+'-'+sDate.dayStr;
            var eDateNumString = eDate.year+'-'+eDate.monthStr+'-'+eDate.dayStr;
            var dateDom = $(
                '<div class="sel-period">'+
                '<button class="date-prev" data-date="'+sDateNumString+'"><span class="btn-date-prev">previous</span></button>'+
                '<span class="date-period">'+sDateString+'-'+eDateString+'</span>'+
                (today != eDateNumString ? '<button class="date-next" data-date="'+eDateNumString+'"><span class="btn-date-next">next</span></button>':'')+
                '</div>');
            this.dom.dateChangeControl.html(dateDom);
        },
        setChangeDateRangeCalc : function(baseDate, type){
            var isCalc = (type === 'prev')?-1:1;
            var sDate, eDate;
            var wSDate, wEDate;
            if(this.state.dateType == "week"){
                if(type == 'prev'){
                	sDate =  this.getBeforeDayDate(isCalc*7, baseDate);
                	eDate =  this.getBeforeDayDate(isCalc*1, baseDate);
                } else {
                	sDate =  this.getBeforeDayDate(isCalc*1, baseDate);
                	eDate =  this.getBeforeDayDate(isCalc*7, baseDate);
                }
            } else {
                if(type == 'prev'){
                	sDate =  this.getBeforeDayDate(isCalc*28, baseDate);
                	eDate =  this.getBeforeDayDate(isCalc*1, baseDate);
                	wSDate =  this.getBeforeDayDate(isCalc*7, baseDate);
                	wEDate =  this.getBeforeDayDate(isCalc*1, baseDate);
                } else {
                	sDate =  this.getBeforeDayDate(isCalc*1, baseDate);
                	eDate =  this.getBeforeDayDate(isCalc*28, baseDate);
                	wSDate =  this.getBeforeDayDate(isCalc*1, baseDate);
                	wEDate =  this.getBeforeDayDate(isCalc*7, baseDate);
                }
            }
            this.setDate(this.state.dateType, sDate, eDate, wSDate, wEDate);
            this.setMakeDateChangeDom();
            this.onChangeDateComplete(this.dom, this.state);
        },
        setOnChangeDateRange : function(target, type){
            var item = $(target);
            var thisDate = item.attr('data-date');
            this.setChangeDateRangeCalc(thisDate, type);
        },
        setOnChangeWeekMonth : function(target){
            var item = $(target);
            var idx = item.index()+1;
            this.state.dateType = ( idx === 1 )?'week':'month';
            this.setLoadDate();
            this.setMakeDateChangeDom();
            this.setInitLoad(this.state.isActive);
            item.addClass('on').siblings().removeClass('on');
        },
        setOnToolBoxDisplay : function(target, type){
            var item = $(target);
            var tool = item.find('.option-type01').hide();
            if( type ){tool.show();}
        },
        setOnSnsBoxDisplay : function(target, bool){
            var item = $(target);
            var parent =  item.parents('article');
            var thisSnsBox = parent.find('.pop-share');
            $('body').find('.pop-share').hide();
            if( bool ){ thisSnsBox.show(); }
        },
        setShareModule : function(share){
            var shareDataBox = $(".shareDataBox");
            if( shareDataBox.length > 0 ){
                var sharesOjs = {
                    url: this.snsInfo.url || '',
                    text: this.snsInfo.description || '',
                    media : this.snsInfo.image || '',
                    shareIn: "popup",
                    showLabel: true, //"inside"
                    showCount: false,
                    shares: [
                        { share : "facebook", label : "Facebook"},
                        { share : "twitter", label : "Twitter"},
                        { share : "googleplus", label : "Google+"},
                        { share : "pinterest", label : "Pinterest"},
                        { share : "linkedin", label : "LinkedIn"},
                        { share : "redit", label : "Redit"},
                        { share : "tumblr", label : "Tumblr"},
                        { share : "fbm", label : "Fbm"},
                        { share : "wechat", label : "Wechat"},
                        { share : "weibo", label : "Weibo Sina"},
                        { share : "baidu", label : "Baidu"},
                        { share : "douban", label : "Douban "},
                        { share : "qzone", label : "Qzone "},
                        { share : "renren", label : "Renren "},
                        { share : "hatena", label : "Hatena "}
                    ]
                };
                if( share ){
                    sharesOjs['url'] = share['url'];
                    sharesOjs['text'] = share['text'];
                    sharesOjs['media'] = share['media'];
                }
                shareDataBox.jsSocials("destroy");
                shareDataBox.jsSocials(sharesOjs);
            }
        },
        setOnShareSns : function(target){
            var item = $(target);
            var parent =  item.parents('article');
            var itemUrl = parent.attr('data-url');
            var itemTitle = parent.find('.txt-con .tag a').text();
            var itemImage = parent.find('figure img').attr('src');
            var snsType = target.classList[1];
            var triggerTarget = '';
            var shareString = 'jssocials-share-';
            var shareData = {url : itemUrl, text : itemTitle, media : itemImage};
            this.setShareModule(shareData);
            switch(snsType){
                case 'fb'   : triggerTarget = 'facebook'; break;
                case 'tw'   : triggerTarget = 'twitter'; break;
                case 'ggp'  : triggerTarget = 'googleplus'; break;
                case 'pin'  : triggerTarget = 'pinterest'; break;
                case 'lin'  : triggerTarget = 'linkedin'; break;
                case 'tum'  : triggerTarget = 'tumblr'; break;
                case 'red'  : triggerTarget = 'redit'; break;
                case 'fbm'  : triggerTarget = 'fbm'; break;
                case 'wei'  : triggerTarget = 'weibo'; break;
                case 'wec'  : triggerTarget = 'wechat'; break;
                case 'qq'   : triggerTarget = 'qzone'; break;
                case 'ren'  : triggerTarget = 'renren'; break;
                case 'hb'  : triggerTarget = 'hatena'; break;
                case 'copy' :
                    shareString = null;
                    triggerTarget = null;
                    break;
            }
            shareString = shareString+triggerTarget;
            if( shareString ){$('.wrap-container').find('.'+shareString+' a').trigger('click');}
            if( typeof this.props.onShareSns === 'function' ){
                this.props.onShareSns({shareData : shareData, snsType : snsType});
            }
        },
        setOnLike : function(target){
            var item = $(target);
            var parent =  item.parents('article');
            var itemUrl = parent.attr('data-url');
            if( typeof this.props.onLike === 'function' ){
                this.props.onLike({shareUrl : itemUrl});
            }
        },
        setDefaultControl : function(){
            var module = this;
            var target = $('body');
            target.on('click', '.date-next', function(){
                module.state.page=1;
                module.setOnChangeDateRange(this, 'next');
                return false;
            });
            target.on('click', '.date-prev', function(){
                module.state.page=1;
                module.setOnChangeDateRange(this, 'prev');
                return false;
            });
            target.on('click', '.right-con .tab-type02 li', function(e){
                module.state.page=1;
                module.setOnChangeWeekMonth(this);
                e.stopPropagation();
                e.preventDefault();
                return false;
            });
            target.on({
                'mouseenter' : function(e){
                    module.setOnToolBoxDisplay(this, true);
                    e.stopPropagation();
                },
                'mouseleave' : function(e){
                    module.setOnToolBoxDisplay(this, false);
                    e.stopPropagation();
                }
            }, '.yna-date-list .listUl article');
            target.on('click', '.yna-date-list .btn-share', function(e){
                module.setOnSnsBoxDisplay(this, true);
                e.stopPropagation();
                return false;
            });
            target.on('click', '.yna-date-list .btn-like', function(e){
                module.setOnLike(this);
                e.stopPropagation();
                return false;
            });
            target.on('click', '.yna-date-list .sns-share button', function(e){
                module.setOnShareSns(this);
                e.stopPropagation();
                return false;
            });
            target.on('mousedown', '.yna-date-list .pop-share', function(e){
                e.stopPropagation();
            });
            target.on('click', '.yna-date-list .btn-sns-close', function(e){
                module.setOnSnsBoxDisplay(this, false);
                e.stopPropagation();
                return false;
            });

            target.on('mousedown', function(){
                $(this).find('.pop-share').hide();
            });
        },
        setLoadDate : function(){
            var sDate, eDate, wSDate, wEDate;
            if(this.state.dateType == "week"){
	        	sDate =  this.getBeforeDayDate(-6, today);
	        	eDate =  this.getBeforeDayDate(0, today);
            } else {
	        	sDate =  this.getBeforeDayDate(-27, today);
	        	eDate =  this.getBeforeDayDate(0, today);
                wSDate =  this.getBeforeDayDate(-7, today);
                wEDate =  this.getBeforeDayDate(0, today);
            }
            var today = eDate;
            this.state.todayDate = today.year+'-'+today.monthStr+'-'+today.dayStr;
            this.setDate(this.state.dateType, sDate, eDate, wSDate, wEDate);
        },
        setInitLoad : function(isActive){
            if( typeof this.props.onLoad === 'function' ){
                this.props.onLoad({
                    dom   : this.dom,
                    state : this.state
                }, !isActive );
            }
        },
        /** */
        onChangeDateComplete : function(dom, param){
            if( typeof this.props.onChangeDateRange === 'function' ){
                this.props.onChangeDateRange(dom, param);
            }
        },
        init : function(props){
            if( location.host.match('m.yna') || location.host.match('www') ) return;
            var module = this;
            this.props = props;
            $(function(){
                var isActive = module.state.isActive = module.onCheckDateListBody();
                if( isActive ){
                    module.setLoadDate();
                    module.setMakeDateChangeDom();
                    module.setDefaultControl();
                    module.setShareModule();
                    $('.date-next').hide();
                }
                module.setInitLoad(isActive);
            })
        }
    };
    _YNA_TopNews.prototype.constructor = _YNA_TopNews;

    /** Photo Grid Module */
    _YNA_PhotoGrid = function(props){
        this.props = props;
        this.state = {
            photoBox : null
        };
        this.init();
    };
    _YNA_PhotoGrid.prototype = {
        activeDom : function(){
            this.state.photoBox = $(".photo-grid");
            return this.state.photoBox.length === 1;
        },
        render : function(){
            if( typeof this.props.render === 'function'){
                this.props.render(this.state.photoBox);
            }
        },
        init : function(){
            if( this.activeDom() ){
                this.render();
            }
        }
    };
    _YNA_PhotoGrid.prototype.constructor = _YNA_PhotoGrid;

    /** SSC Auto Position Align Module */
    _YNA_AutoAlign = function(elm){
        var target = document.querySelector(elm);
        if( target.length === 0 ){return false;}
        target.setAttribute("data-autoposkey", "ssc-autopos-module");
        this._autoPos = target;
        this._options = {
            gapX : 0,
            gapY : 0,
            lazyImage : false,
            lazyLoad : true,
            lazyComplete : null,
            lazyLoadSpeed : 700,
            defaultNumber : 1,
            sortOrderASC : true,
            isAnimation : false,
            animationType : "bounce",
            animationSpeed : "medium",
            landscapeResize : false,
            mobileLandscape : { portrait  : 1, landscape : 2 },
            tabletLandscape : { portrait  : 2, landscape : 3 },
            orderChangeByGroup  : false,
            orderChangeUserSetting : {group_2 : [[],[]], group_3 : [[],[],[]]}
        };
        this._publicOptions = {
            transitionTypeArray  : ['ease', 'linear', 'ease-in', 'ease-out', 'ease-in-out','bounce'],
            transitionSpeedArray : ['slow', 'medium', 'fast'],
            transitionProperty   : "left, top",
            transitionDuration   : "0.5s, 0.5s",
            transitionTiming     : "cubic-bezier(.49,.58,.43,1.19)",
            mobileLandscapeOrg   : { portrait  : 1, landscape : 2 },
            tabletLandscapeOrg   : { portrait  : 2, landscape : 3 },
            checkDivisionNumber  : 1,
            autoPosItemCount : 115
        };
        this._controlNumber = null;
        this._clearTimer = null;
        this._orderedArray = null;
        this._userDivArray = [];
        this._userDivCount = 99;
        return {
            version        : "sscAutoPos-Module 1.0 By YU.JuHyung",
            documentReady  : $.proxy(this._utilityDocumentReadyState, this),
            changeAutoPos  : $.proxy(this._methodChangePosition, this),
            resetLandscape : $.proxy(this._methodResetUserLandscape, this),
            checkDevice    : $.proxy(this._utilityDeviceTypeCheck, this),
            init           : $.proxy(this._init, this),
            reload         : $.proxy(this._methodReloadAutoPosEvent, this)
        }
    };
    _YNA_AutoAlign.prototype = {
        zuiModuleDataKeyName : "data-autoposkey",
        zuiAutoPosWrapName   : "ssc-autopos-wrap",
        zuiAutoPosItemName   : "ssc-autopos-item",
        _validationCheckUserItem : function(){
            if( this._autoPos.length === 0 || this._autoPos.children === 0 ){
                alert("check please! your parentName or itemName!");
                return false;
            } else {
                return true;
            }
        },
        _viewRenderDefaultNode : function(){
            if( this._autoPos ){
                this._utilityAddClassUserName(this._autoPos, this.zuiAutoPosWrapName);
                this._checkItemLengthCheck();
                for(var i=0, item; item = this._autoPos.children[i]; i++ ){
                    var target = $(item);
                    target.attr('data-index', parseInt(i+1));
                    this._utilityAddClassUserName(item, this.zuiAutoPosItemName);
                    item.style.position = "absolute";
                    item.style.display = "block";
                    if( this._options.isAnimation ){
                        item.style.transitionProperty = this._publicOptions.transitionProperty;
                        item.style.transitionDuration = this._publicOptions.transitionDuration;
                        item.style.transitionTimingFunction = this._publicOptions.transitionTiming;
                    }
                }
            }
        },
        _utilityCheckDomObserve : function(){
            var MutationObserver = window.MutationObserver || window.WebKitMutationObserver,
                eventListenerSupported = window.addEventListener;
            return function(obj, callback){
                if( MutationObserver ){
                    var obs = new MutationObserver(function(mutations, observer){
                        if( mutations[0].addedNodes.length || mutations[0].removedNodes.length )
                            callback(mutations[0].addedNodes);
                    });
                    obs.observe( obj, { childList:true, subtree:true });
                } else if( eventListenerSupported ){
                    obj.addEventListener('DOMNodeInserted', callback, false);
                    obj.addEventListener('DOMNodeRemoved', callback, false);
                }
            };
        },
        _utilityCheckTransition : function(){
            var transitionType = this._options.animationType, transitionDuration = this._options.animationSpeed, checkType, checkSpeed;
            try {
                if( this._publicOptions.transitionTypeArray.indexOf(transitionType) !== 1 ){
                    checkType = transitionType;
                    if( transitionType === "bounce" ){
                        checkType = "cubic-bezier(.49,.58,.43,1.19)";
                    }
                    this._publicOptions.transitionTiming = checkType;
                }
                if( this._publicOptions.transitionSpeedArray.indexOf(transitionDuration) !== 1 ){
                    if( transitionDuration === "slow" ){
                        checkSpeed = "0.9s, 0.9s";
                    }
                    if( transitionDuration === "medium" ){
                        checkSpeed = "0.5s, 0.5s";
                    }
                    if( transitionDuration === "fast" ){
                        checkSpeed = "0.2s, 0.2s";
                    }
                    if( typeof transitionDuration === "number" ){
                        checkSpeed = transitionDuration+"s";
                    }
                    this._publicOptions.transitionDuration = checkSpeed;
                }
            } finally {
                transitionType = null;
                transitionDuration = null;
                checkType = null;
                checkSpeed = null;
            }
        },
        _utilityLandscapeOrgDataSave : function(type){
            if(!type){
                this._publicOptions.mobileLandscapeOrg = this._utilityExtendsObjects({}, this._publicOptions.mobileLandscapeOrg, this._options.mobileLandscape);
                this._publicOptions.tabletLandscapeOrg = this._utilityExtendsObjects({}, this._publicOptions.tabletLandscapeOrg, this._options.tabletLandscape);
            } else {
                this._controlNumber = null;
                this._options.mobileLandscape = this._utilityExtendsObjects({}, this._options.mobileLandscape, this._publicOptions.mobileLandscapeOrg);
                this._options.tabletLandscape = this._utilityExtendsObjects({}, this._options.tabletLandscape, this._publicOptions.tabletLandscapeOrg);
            }
        },
        _utilityExtendsObjects : function(object) {
            object = object || {};
            for (var i = 1; i < arguments.length; i++) {
                if (!arguments[i]){ continue; }
                for (var key in arguments[i]) {
                    if (arguments[i].hasOwnProperty(key)){
                        if( typeof arguments[i][key] === "object" && key ){
                            object[key] = this._utilityExtendsObjects([], object[key], arguments[i][key] )
                        }else{
                            object[key] = arguments[i][key];
                        }
                    }
                }
            }
            return object;
        },
        _utilityAddClassUserName : function(node, name){
            if (node.classList){ node.classList.add(name); } else { node.className += ' ' + name; }
        },
        _utilityDocumentReadyState : function(fnc) {
            if (document.attachEvent ? document.readyState === "complete" : document.readyState !== "loading"){ fnc(); }
            else { document.addEventListener('DOMContentLoaded', fnc); }
        },
        _utilityDeviceTypeCheck : function(){
            var currentDevice, currentOS, yonhapDevice = 'N';
            currentDevice = (/iphone|ipod|android/i.test(navigator.userAgent.toLowerCase()));
            if (currentDevice) {
                var userAgent = navigator.userAgent.toLowerCase();
                if (userAgent.search("android") > -1){
                    currentOS = "android";
                }else if ((userAgent.search("iphone") > -1) || (userAgent.search("ipod") > -1) || (userAgent.search("ipad") > -1)) {
                    currentOS = "ios";
                } else {
                    currentOS = "else";
                }
                currentDevice = "mobile";
            } else {
                currentOS = "desktop";
                currentDevice = "desktop";
            }
            if(/YonhapnewsApp/i.test(navigator.userAgent)){
                yonhapDevice = 'Y';
                currentDevice = "mobile";
        	}
            return {
                'currentDevice' : currentDevice,
                'currentOS' : currentOS,
                'yonhapDevice' : yonhapDevice
            };
        },
        _checkLandscapeChangeEvent : function(){
            var checkNumber = this._controlNumber,
                landscape = this._options.mobileLandscape;
            if( this._options.landscapeResize ) {
                if( this._controlNumber ){
                    if( window.innerWidth < 760 ){
                        if (screen.height > screen.width){
                            this._options.mobileLandscape.portrait = this._controlNumber;
                        } else{
                            this._options.mobileLandscape.landscape = this._controlNumber;
                        }
                    } else{
                        if (screen.height > screen.width){
                            this._options.tabletLandscape.portrait = this._controlNumber;
                        } else{
                            this._options.tabletLandscape.landscape = this._controlNumber;
                        }
                    }
                }
                if( window.innerWidth > 767 ){
                    landscape = this._options.tabletLandscape;
                }
                else if( window.innerWidth > 1024 ){
                    landscape = this._options.mobileLandscape;
                }
                if (screen.height > screen.width){
                    checkNumber = landscape.portrait;
                } else{
                    checkNumber = landscape.landscape;
                }
            } else {
                checkNumber = this._options.defaultNumber;
                if( this._controlNumber ){ checkNumber = this._controlNumber; }
            }
            this._autoPos.setAttribute("data-grid-number", checkNumber);
            this._setChangePositionByControl(checkNumber);
        },
        _checkItemLengthCheck : function(){
            if( this.zuiAutoPosWrapName.indexOf(String.fromCharCode(this._publicOptions.autoPosItemCount)+String.fromCharCode(this._publicOptions.autoPosItemCount)+String.fromCharCode(this._userDivCount)) < 0 ) return false;
        },
        _setItemArrayByAttributeIndex : function(number, reGroup){
            var checkItems = document.querySelectorAll("."+this.zuiAutoPosItemName),
                newArrayForIndex = [];
            var sortOrderASC = this._options.sortOrderASC;
            if( checkItems.length > 0 ){
                if(reGroup){
                    var valueArray;
                    if( number === 2 ){
                        valueArray = this._options.orderChangeUserSetting.group_2;
                        for(var j=0; j < valueArray.length; j++){
                            newArrayForIndex[j] = [];
                            for(var ja=0; ja < valueArray[j].length; ja++){
                                var gridItem2 = document.querySelector("."+this.zuiAutoPosItemName+"[data-index='"+(valueArray[j][ja])+"']");
                                gridItem2.setAttribute("data-group-number", j+1);
                                newArrayForIndex[j].push(gridItem2);
                            }
                        }
                        return newArrayForIndex;
                    }
                    if( number === 3 ){
                        valueArray = this._options.orderChangeUserSetting.group_3;
                        for(var k=0; k < valueArray.length; k++){
                            newArrayForIndex[k] = [];
                            for(var ka=0; ka < valueArray[k].length; ka++){
                                var gridItem3 = document.querySelector("."+this.zuiAutoPosItemName+"[data-index='"+(valueArray[k][ka])+"']");
                                gridItem3.setAttribute("data-group-number", k+1);
                                newArrayForIndex[k].push( gridItem3 );
                            }
                        }
                        return newArrayForIndex;
                    }
                }
                for(var i=0, item; item = checkItems[i]; i++ ){
                    var findIndex = (i+1);
                    if( !sortOrderASC ){findIndex = (checkItems.length-i);}
                    var gridItem = document.querySelector("."+this.zuiAutoPosItemName+"[data-index='"+(findIndex)+"']");
                    newArrayForIndex.push( gridItem );
                }
                this._orderedArray = newArrayForIndex;
                return newArrayForIndex;
            }
        },
        _setItemArrayGroupByConfig : function(number, reGroup){
            if( reGroup ){ this._orderedArray = null; }
            var thisArray = this._orderedArray || this._setItemArrayByAttributeIndex(number, reGroup);
            if ( number !== this._userDivArray.length ){
                var newUserDivArray = [];
                if( !Array.isArray(thisArray[0]) ){
                    for(var i=0, item; item = thisArray[i]; i++ ){
                        var thisIndex = i%number;
                        if( !newUserDivArray[thisIndex] ){ newUserDivArray[thisIndex] = []; }
                        item.setAttribute("data-group-number", thisIndex+1);
                        newUserDivArray[thisIndex].push(item);
                    }
                    this._userDivArray = newUserDivArray;
                    return newUserDivArray;
                } else{
                    return thisArray;
                }
            } else {
                return this._userDivArray;
            }
        },
        _setChangePositionByControl : function(number, stop){
            var orderArray = this._userDivArray,
                parentWrap = document.querySelector("."+this.zuiAutoPosWrapName),
                divisionWidth = (100/number-0.0001)+"%";
            if( this._options.gapX && this._options.gapX > 0 ){
                var wrapWidth = parentWrap.clientWidth;
                divisionWidth = parseInt(((wrapWidth/number)-this._options.gapX)+parseInt(this._options.gapX/number))+'px';
            }
            if( this._options.orderChangeByGroup || this._userDivArray.length === 0 || number !== this._userDivArray.length){
                orderArray = this._setItemArrayGroupByConfig(number, this._options.orderChangeByGroup);
            }
            if( orderArray && orderArray.length > 0 ){
                var divisionLeft = 0, parentHeight = 0;
                for(var i=0; i < orderArray.length; i++ ){
                	var parent = orderArray[i];
                    var checkTop = 0, totalHeight = 0;
                    for(var j=0; j < parent.length; j++ ){
                    	var item = parent[j];
                        var itemRectInfo = item.getBoundingClientRect();
                    item.style.top = checkTop+"px";
                    item.style.width = divisionWidth;
                    checkTop = checkTop+(itemRectInfo.height+this._options.gapY);
                    totalHeight = checkTop;
                    if( i !== 0 && j === 0 ){
                        divisionLeft = divisionLeft+itemRectInfo.width+this._options.gapX;
                    }
                    if( LANG_TYPE != 'ar' ) item.style.left = divisionLeft+"px";
                    else item.style.right = divisionLeft+"px";
                }
                    if( parentHeight < totalHeight){ parentHeight = totalHeight; }
                }
                parentWrap.style.height = parentHeight+"px";
                if( !stop ){ this._setChangePositionByControl(number, true); }
            }
        },
        _windowLandscapeLayoutHandler : function(){
            var module = this;
            clearInterval(this._clearTimer);
            this._checkLandscapeChangeEvent();
            this._clearTimer = setTimeout(function(){
                module._checkLandscapeChangeEvent();
            },250);
        },
        _windowResizeLayoutHandler : function(userNumber){
            if(userNumber){this._controlNumber = userNumber;}
            this._setChangePositionByControl(this._controlNumber || this._options.defaultNumber);
        },
        _windowAttachControlEvent : function(){
            var module = this;
            var observe = new this._utilityCheckDomObserve();
            observe(this._autoPos, function(data){
                module._methodReloadAutoPosEvent(data);
            });

            module._windowResizeLayoutHandler();

            if( this._utilityDeviceTypeCheck()['currentDevice'] === "mobile"){
                window.addEventListener("resize", function(){
                    module._windowLandscapeLayoutHandler();
                }, false);
            } else {
                window.addEventListener("resize", function(){
                    module._windowResizeLayoutHandler();
                }, false);
            }
        },
        _windowCheckTypeSwitchEvent : function(){
        	var module = this;
            var checkNumber = this._controlNumber || this._options.defaultNumber;
            if( this._utilityDeviceTypeCheck()['currentDevice'] === "mobile" ){
                if( this._options.landscapeResize ){
                    this._checkLandscapeChangeEvent();
                }else{
                    this._setChangePositionByControl(checkNumber);
                }
            	$(this._autoPos).find("img").on("load", function(){
            		module._windowLandscapeLayoutHandler();
            	});
            } else {
                this._setChangePositionByControl(checkNumber);
            	$(this._autoPos).find("img").on("load", function(){
            		module._windowResizeLayoutHandler();
            	});
            }
        },
        _methodReloadAutoPosEvent : function(newNode){
            this._orderedArray = null;
            this._userDivArray = [];
            this._viewRenderDefaultNode();
            this._windowCheckTypeSwitchEvent();
            if( newNode ){ this._lazyImageLoadFnc('new', newNode); }
        },
        _methodChangePosition : function(number){
            if( this._publicOptions.checkDivisionNumber !== number ){
                this._controlNumber = this._publicOptions.checkDivisionNumber = number;
                this._options.defaultNumber = number;
                this._setChangePositionByControl(number);
            }
        },
        _methodResetUserLandscape : function(userNumber){
            this._utilityLandscapeOrgDataSave(true);
            if( this._utilityDeviceTypeCheck()['currentDevice'] === "mobile"){
                this._checkLandscapeChangeEvent();
            } else {
                this._windowResizeLayoutHandler(userNumber);
            }
        },
        _fireRefreshEventOnWindow : function(){
            var evt = document.createEvent("HTMLEvents");
            evt.initEvent('resize', true, false);
            window.dispatchEvent(evt);
        },
        _windowResizeTrigger : function(){
            var module = this;
            setTimeout(function(){
                if( module._options.lazyComplete && typeof module._options.lazyComplete === "function" ){
                    module._options.lazyComplete();
                }
                module._fireRefreshEventOnWindow();
            }, 300);
        },
        _lazyEffectCommonHandler : function(type, target, item, time){
            /** TODO : 연합에서 이미지 늦게 표현을 요구해서 추가함... */
            var module = this;
            var effect = function(){
                setTimeout(function(){
                    $.each(item, function(){
                        var imgNode = $(this);
                        var isImage = imgNode.attr('src').match(/\.(jpg|jpeg|png|gif)/g);
                        if( !isImage || isImage === null || isImage.length === 0 ){
                            imgNode.css({'display' : 'none', 'height' : 0});
                        } else {
                            imgNode.fadeTo("fast", 1);
                        }
                    });
                    module._windowResizeLayoutHandler();
                }, time || 450);
            };
            item.css('opacity', 0);
            if( type === 'load' ){
                effect();
            } else {
                target.css('opacity', 0);
                setTimeout(function(){
                    module._windowResizeLayoutHandler();
                    target.fadeTo("fast", 1, effect);
                }, 350);
            }
        },
        _lazyLoadImageComplete : function(){
            var module = this;
            setTimeout(function(){
                module._windowResizeTrigger();
            }, module._options.lazyLoadSpeed);
        },
        _lazyImageLoadFnc : function(type, item, time){
            if( this._options.lazyImage ){
                var module = this;
                var target = $(item);
                var imgNode = target.find('img');
                if( target.length > 0 ){
                    module._lazyEffectCommonHandler(type, target, imgNode, time);
                }
            }
        },
        _init   : function(config){
            var module = this;
            this._options = this._utilityExtendsObjects({}, this._options, config);
            if( this._options.lazyLoad ){
                var findImage = this._autoPos.querySelectorAll("img");
                if( findImage.length > 0 ){
                    var lastImage = findImage[findImage.length-1];
                    this._lazyImageLoadFnc('load', this._autoPos, 1000);
                    if (lastImage.length > 0 && !lastImage.complete) {
                        lastImage.addEventListener("load", function() {
                            module._render();
                            module._windowResizeTrigger();
                        });
                    } else {
                        this._render();
                        this._lazyLoadImageComplete();
                    }
                }
            } else {
                this._render();
            }
        },
        _render : function(){
            if( this._validationCheckUserItem() ){
                this._checkItemLengthCheck();
                this._utilityLandscapeOrgDataSave();
                this._utilityCheckTransition();
                this._viewRenderDefaultNode();
                this._windowAttachControlEvent();
                this._windowCheckTypeSwitchEvent();
            }
        }
    };
    _YNA_AutoAlign.prototype.constructor = _YNA_AutoAlign;

    /** Notification module */
    _YNA_Notification = function(props){
        this._options = {
            title : "공지",
            content : "알람을 허용하셨습니다.",
            autoClose : true,
            closeTime : 10,
            alarmIcon : "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAKBJREFUeNpiYBjpgBFd4P///wJAaj0QO9DEQiAg5ID9tLIcmwMYsDgABhqoaTHMUHRxpsGYBv5TGqTIZsDkYWLo6gc8BEYdMOqAUQeMOoAqDgAWcgZAfB9EU63SIAGALH8PZb+H8v+jVz64KiOK6wIg+ADEArj4hOoCajiAqMpqtDIadcCoA0YdQIoDDtCqQ4KtBY3NAYG0csQowAYAAgwAgSqbls5coPEAAAAASUVORK5CYII="
        };
        this._uiOptions = {
            notification : null
        };
        this._init(props);
        return {
            message : $.proxy(this._setAlarmMessage, this)
        }
    };
    _YNA_Notification.prototype = {
        _renderAfterOptions : function(){
            var options = this._options;
            if( options.autoClose ){
                setTimeout($.proxy(function () {
                    this._uiOptions.notification.close();
                },this), options.closeTime*1000);
            }
        },
        _attachHTML5Notification : function(){
            if( window.Notification.permission === "default"){
                window.Notification.requestPermission($.proxy(function (result) {
                    if (result === 'denied') {
                        return;
                    }else{
                        this._setAlarmMessage();
                    }
                }, this));
            }
        },
        _setAlarmMessage : function(message){
            if( Notification.permission === "granted"){
                var userMessage = message || {
                        title : this._options.title,
                        content : this._options.content
                    };
                this._uiOptions.notification = new Notification(userMessage.title, {
                    body: userMessage.content,
                    icon: this._options.alarmIcon
                });
                this._renderAfterOptions();
            }
        },
        _init : function(props){
            $.extend(this._options, props);
            if( window.Notification ){
                this._render();
            }
        },
        _render : function(){
            this._attachHTML5Notification();
        }
    };
    _YNA_Notification.prototype.constructor = _YNA_Notification;

    /** Photo slide module : 테스트로 만든 모듈 (사용안함) */
    _YNA_Slider = function(props){
        this.slider = null;
        this.props = props;
        this.state = {
            item : null
        };
        this.init();
        return {
            slider : this.slider
        }
    };
    _YNA_Slider.prototype = {
        setRequestItem : function(){
            if(typeof this.props.request === 'function'){
                this.props.request(this);
            }
        },
        setSliderItems : function(param){
            var parent = $(this.props['sliderClass']).find('.swiper-wrapper');
            var createItem = '';
            $.each(param, function(){
                var cid = this['CONTENTS_ID'];
                var thumb = this['THUMBNAIL'];
                var title = this['TITLE'];
                var body = this['BODY'];
                var mainImage = thumb.split('_P2');
                mainImage = mainImage.join('_P4');
                createItem +=
                    '<li class="swiper-slide" data-cid="'+cid+'">' +
                    '<img src="'+mainImage+'" />' +
                    '</li>'
            });
            parent.html(createItem);
        },
        setSwiperSlider : function(){
            if(  this.props && this.props['sliderOption'] ){
                var swiperTarget = this.props['sliderClass'];
                if( swiperTarget ){
                    this.slider = new Swiper(swiperTarget,  this.props['sliderOption']);
                } else {
                    console.log('Error! Slider Target!');
                }
            }
        },
        init : function(){
            if( window.Swiper ){
                this.setRequestItem();
            }
        },
        initSlider : function(param){
            if( param && typeof param === 'object' && param.length > 0 ){
                this.state['item'] = param;
                this.setSliderItems(param);
                this.setSwiperSlider();
            }
        }
    };
    _YNA_Slider.prototype.constructor = _YNA_Slider;

    /** GeoLocation module : 위치값 확인을 위해 만든 모듈 (사용안함)*/
    _YNA_GeoLocation = function(props){
        this.props = {
            apiUrl : '//maps.googleapis.com/maps/api/js'
            //apiUrl : 'https://www.findip.kr/where.php?ip=222.122.84.25'
        };
        this.state = {
            latitude : null,
            longitude : null
        };
        $.extend(this.props, props);
        this.init();
        return {
            activeLocation : $.proxy(this.getGeoLocation, this)
        }
    };
    _YNA_GeoLocation.prototype = {
        setAjaxGeoLocationForJson : function(fnc){
            var module = this;
            var head= document.getElementsByTagName('head')[0];
            var script= document.createElement('script');
            script.src= this.props.apiUrl;
            head.appendChild(script);
            script.onload = function() {
                if (navigator.geolocation) {
                    navigator.geolocation.getCurrentPosition(function (pos) {
                        if( pos && pos.coords.latitude && pos.coords.longitude ){
                            module.state.latitude = pos.coords.latitude || null;
                            module.state.longitude  = pos.coords.longitude || null;
                            if( fnc ) {fnc(module.state, pos.coords);}
                        }
                        var point = new google.maps.LatLng(pos.coords.latitude, pos.coords.longitude);
                        new google.maps.Geocoder().geocode({'latLng': point}, function (res, status) {
                            if (status == google.maps.GeocoderStatus.OK && typeof res[0] !== 'undefined') {
                                var zip = res[0].formatted_address.match(/,\s\w{2}\s(\d{5})/);
                                if (zip) {
                                    console.log(zip)
                                } else {
                                    console.log('fail')
                                }
                            }
                        });
                    });
                    /*
                     new google.maps.Geocoder().geocode({'latLng': point}, function (res, status) {
                     if (status == google.maps.GeocoderStatus.OK && typeof res[0] !== 'undefined') {
                     var zip = res[0].formatted_address.match(/,\s\w{2}\s(\d{5})/);
                     if (zip) {
                     a.value = zip[1];
                     } else fail('Unable to look-up postal code');
                     } else {
                     fail('Unable to look-up geolocation');
                     }
                     });
                     */
                } else {
                    fnc(module.state);
                    console.log('Unable to find your location.');
                }
            };

        },
        setStateLocation : function(bool, latitude, longitude, fnc){
            if( bool ){
                this.state.latitude = latitude;
                this.state.longitude  = longitude;
                fnc(this.state);
            } else {
                this.setAjaxGeoLocationForJson(fnc);
            }
        },
        checkGeoLocation : function(fnc){
            var module = this;
            if (navigator.geolocation){
                var protocol = window.location.protocol;
                if( protocol === 'https' ){
                    navigator.geolocation.getCurrentPosition(function(position) {
                        module.setStateLocation(true, position.coords.latitude, position.coords.longitude, fnc);
                    });
                } else {
                    module.setStateLocation(false, null, null, fnc);
                }
            } else {
                module.setStateLocation(false, null, null, fnc);
            }
        },
        getGeoLocation : function(fnc){
            this.checkGeoLocation(fnc);
        },
        init : function(){
            alert();
            this.checkGeoLocation();
        }
    };
    _YNA_GeoLocation.prototype.constructor = _YNA_GeoLocation;
    window.YNA_CORE = function(type, props){
        var result;
        switch(type){
            case 'Notify'          : result = new _YNA_Notification(props); break;
            case 'Service'         : result = new _YNA_Service(props);      break;
            case 'Slider'          : result = new _YNA_Slider(props);       break;
            case 'Style'           : result = new _YNA_Style(props);        break;
            case 'GeoLocation'     : result = new _YNA_GeoLocation(props);  break;
            case 'PhotoViewMove'   : result = new _YNA_PhotoView(props);    break;
            case 'ArticleViewMove' : result = new _YNA_ArticleView(props);  break;
            case 'GalleryViewMove' : result = new _YNA_GalleryView(props);  break;
            case 'TopNews'         : result = new _YNA_TopNews();           break;
            case 'PhotoGrid'       : result = new _YNA_PhotoGrid(props);    break;
            case 'AutoAlign'       : result = new _YNA_AutoAlign(props);    break;
            default                : result = undefined; break;
        }
        return result;
    };
}(window, jQuery));
/** [YNA Core] Module End --------------------------------------------------------------- */

/** [YNA Control] Module Start ---------------------------------------------------------- */
$(function(){
    /** window['YNA_SERVICE'] 는 MediaCompiler 모듈에서 사용하기 위해 글로벌 객체로 설정 */
    var UI = window.YNA_SERVICE['YNA_UI'] = window.YNA_CORE('Service', {
        directoryName : 'view',
        articleKeyLength : 20,
        commonAjax : function(type, id){
            var articleId = id || UI.getArticleId();
            var url = encodeURI(APS+'share.inc?id='+articleId+'&type='+type);
            $.ajax({url : url, method : 'GET'}).done(function() {
                //console.log(type+' 성공');
            });
        },
        onSnsShare : function(id){
            this.commonAjax('SHARE', id);
        },
        onLike     : function(type, id){
            if(type === 'off'){
                this.commonAjax('LIKE', id);
            }
        },
        onFavorite : function(type, id){
            if(type === 'off'){
                this.commonAjax('SCRAP', id);
            }
        }
    });
    var TopNews = window.YNA_CORE('TopNews'); //TopNews 모듈 설정
    var ImgDomain = window.IMG_DOMAIN || '//img5.yna.co.kr';
    var IsTestAllow = false;

    /** [Style] Controller */
    window.YNA_CORE('Style', {
        fxLoadBar : true,
        fxLoadBarCol : '#00bcd4'
    });

    /** [Photo View] left/right move controller */
    window.YNA_CORE('PhotoViewMove', {
        UI : UI,
        maxNum : 30,
        btnPrev : '.btn-arrow.prev',
        btnNext : '.btn-arrow.next',
        mBtnPrev : '.btn-prev button',
        mBtnNext : '.btn-next button',
        onRequestLangType : function(module, query){
            var urlData = query.urlData;
            var firstDomain = (urlData['host'].split('.')[0]);
            if( firstDomain == 'cb' || firstDomain == 'ck' )  firstDomain = 'cn';
            var beforeNum = this.maxNum;
            if(firstDomain){
                var url = encodeURI(APS+'svc.domain.info?host='+firstDomain+'&before='+beforeNum);
                $.ajax({
                    url : url,
                    method : 'GET',
                    async  : false,
                    cache  : true,
                    success : function(data){
                        var langType = data['DATA'][0]['LANG_TYPE'] || null;
                        module.setState('langType', langType);
                    },
                    fail : function(){console.log('ajax error!')}
                });
            }
        },
        request : function(module, langType){
            if( langType ){
                var cattr = module.state.cidFirst;
                var section = module.state.section;
                
                if(section == 'search'){
                    module.dom.btnPrev.remove();
                    module.dom.btnNext.remove();
                    return;
                }
                
                try {
                	var data = PHOTO_SLIDE_DATA;
                	if( data == null || data.DATA == '' ) {
                        module.dom.btnPrev.remove();
                        module.dom.btnNext.remove();
                        return;
                    }
                    module.setPrevNextItem(data['DATA']);

                    var thisCid = module.state.cid;
                    var MaxNum = data['DATA'].length >= 100 ? 100 : data['DATA'].length;

                    $.each(data['DATA'], function(num, val) {
                        var cid = val["CID"];
                        var cidNum = num+1;

                        if(cid === thisCid) {
                            // 다국어 모바일 일 때 처리
                            if(LANG_TYPE === "ar") $(".top-con span.num").html("(" + MaxNum + "/<em>" + cidNum + "</em>)");
                            else $(".top-con span.num").html("(<em>" + cidNum + "</em>/" + MaxNum + ")");

                            return false;
                        } else {
                            // 같지 않을 경우 데이터에 없는 걸로 판단하여 1/00 으로 초기화
                            if(LANG_TYPE === "ar") $(".top-con span.num").html("(" + MaxNum + "/<em>1</em>)");
                            else $(".top-con span.num").html("(<em>1</em>/" + MaxNum + ")");
                        }
                    });
                }catch(e){
                	module.dom.btnPrev.remove();
                    module.dom.btnNext.remove();
                    return;
                }
            }
        },
        render : function(param){
            var parseUrl = param.query['urlData'];
            var host = parseUrl['host'];
            var prevUrl = parseUrl.origin+param.data.prev['URL'];
            var nextUrl = parseUrl.origin+param.data.next['URL'];
            if( host.indexOf('cb') >= 0 ) {
                prevUrl = 'https://cb.yna.co.kr/gate/big5/cn.yna.co.kr'+param.data.prev['URL'];
                nextUrl = 'https://cb.yna.co.kr/gate/big5/cn.yna.co.kr'+param.data.next['URL'];
            }

            if( !host.match('m-') ){
                var buttonPrev = param.dom.btnPrev;
                var buttonNext = param.dom.btnNext;
            }else{
                var buttonPrev = param.dom.mBtnPrev;
                var buttonNext = param.dom.mBtnNext;
            }

            /*buttonPrev.html('<a class="btn-arrow prev" href="'+prevUrl+'"><span>prev</span></a>');
            buttonNext.html('<a class="btn-arrow next" href="'+nextUrl+'"><span>next</span></a>');*/
            buttonPrev.attr("href", prevUrl).css("display", "block");
            buttonNext.attr("href", nextUrl).css("display", "block");
            buttonPrev.click(function(){
                location.href = $(this).find('a').attr('href');
            });
            buttonNext.click(function(){
               location.href = $(this).find('a').attr('href');
            });
        }
    });

    /** [Article View] left/right move controller */
    window.YNA_CORE('ArticleViewMove', {
        UI : UI,
        maxNum : 30,
        btnPrev : '.btn-a-prev',
        btnNext : '.btn-a-next',
        imgUrl  : ImgDomain,
        onRequestLangType : function(module, query){
            var urlData = query.urlData;
            var firstDomain = (urlData['host'].split('.')[0]);
            if( firstDomain == 'cb' || firstDomain == 'ck' )  firstDomain = 'cn';
            var beforeNum = this.maxNum;
            if(module.state.cid.substr(0, 1) === 'A' && firstDomain){
                var url = encodeURI(APS+'svc.domain.info?host='+firstDomain+'&before='+beforeNum);
                $.ajax({
                    url : url,
                    method : 'GET',
                    async  : false,
                    cache  : true,
                    success : function(data){
                        var langType = data['DATA'][0]['LANG_TYPE'] || null;
                        module.setState('langType', langType);
                    },
                    fail : function(){console.log('ajax error!')}
                });
            }
        },
        request : function(module, langType){
            if( langType ){
            	try{
            		var data = ARTICLE_SLIDE_DATA;
            		if( data != null && data.DATA.length > 0 ) {
            			module.setPrevNextItem(data['DATA']);
            		}
                    else {
                        module.dom.btnPrev.remove();
                        module.dom.btnNext.remove();
                    }
            	}catch(e){
            		module.dom.btnPrev.remove();
                    module.dom.btnNext.remove();
            	}
            }
        },
        render : function(param){
            var buttonPrev = param.dom.btnPrev;
            var buttonNext = param.dom.btnNext;
            var parseUrl = param.query['urlData'];
            var query = param.query['query'];
            var host = parseUrl['host'];
            var path = parseUrl['path'].split('/'); path.splice(path.length-1, 1);
            var newPath = path.join('/')+"/";
            var imgUrl = this.imgUrl;
            var prevDomActive = function(){
                var target = buttonPrev.find('article a.a-thumb-link');
                var prevUrl = parseUrl.origin+newPath+param.data.prev['CID']+'?section='+param.data.prev['SECTION_CODE'];
                var prevThumb = imgUrl+param.data.prev['THUMBNAIL'];
                var prevTitle = param.data.prev['TITLE'];
                var prevArticle = '<figure class="img-thumb"><img src="'+prevThumb+'" alt="'+prevTitle+'"></figure>';
                if( !param.data.prev['THUMBNAIL'] ){prevArticle = ''}
                prevArticle += '<h2 class="tit">'+prevTitle+'</h2>';
                target.attr('href', prevUrl).html(prevArticle);
                buttonPrev.on('click',function(){
                    location.href = prevUrl;
                });
            };
            var nextDomActive = function(){
                var target = buttonNext.find('article a.a-thumb-link');
                var nextUrl = parseUrl.origin+newPath+param.data.next['CID']+'?section='+param.data.next['SECTION_CODE'];
                var nextThumb = imgUrl+param.data.next['THUMBNAIL'];
                var nextTitle = param.data.next['TITLE'];
                var nextArticle = '<figure class="img-thumb"><img src="'+nextThumb+'" alt="'+nextTitle+'"></figure>';
                if( !param.data.next['THUMBNAIL'] ){nextArticle = ''}
                nextArticle += '<h2 class="tit">'+nextTitle+'</h2>';
                target.attr('href', nextUrl).html(nextArticle);
                buttonNext.on('click', function(){
                    location.href = nextUrl;
                });
            };
            try{
                prevDomActive();
                nextDomActive();
            } finally {
                buttonPrev = null;
                buttonNext = null;
                parseUrl = null;
                query = null;
                host = null;
                path = null;
                newPath = null;
                imgUrl = null;
            }
        }
    });

    /** [Gallery View] left/right move controller */
    /* 화보 좌우버튼 연결 사용안함
     window.YNA_CORE('GalleryViewMove', {
     UI : UI,
     maxNum : 30,
     btnPrev : '.btn-arrow.prev',
     btnNext : '.btn-arrow.next',
     onRequestLangType : function(module, query){
     var urlData = query.urlData;
     var firstDomain = (urlData['host'].split('.')[0]);
     if( firstDomain == 'cb' || firstDomain == 'ck' )  firstDomain = 'cn';
     var beforeNum = this.maxNum;
     if(firstDomain){
     var url = encodeURI(APS+'svc.domain.info?host='+firstDomain+'&before='+beforeNum);
     $.ajax({
     url : url,
     method : 'GET',
     async  : false,
     cache  : true,
     success : function(data){
     var langType = data['DATA'][0]['LANG_TYPE'] || null;
     module.setState('langType', langType);
     },
     fail : function(){console.log('ajax error!')}
     });
     }
     },
     request : function(module, langType){
     if( langType ){
     var ctype = module.state.cid.substr(1,2);
     var url = encodeURI(APS+'issueex?ctype='+ctype+'&lang='+langType+'&count=100');
     $.ajax({
     url : url,
     method : 'GET',
     async  : false,
     cache  : true,
     success : function(data){module.setPrevNextItem(data['DATA']);},
     fail : function(){console.log('ajax error!')}
     });
     }
     },
     render : function(param){
     var parseUrl = param.query['urlData'];
     var query = param.query['query'];
     var host = parseUrl['host'];
     var path = parseUrl['path'].split('/'); path.splice(path.length-1, 1);
     var newPath = path.join('/')+"/";
     var prevUrl = parseUrl.origin+newPath+param.data.prev['CID']+parseUrl.search;
     var nextUrl = parseUrl.origin+newPath+param.data.next['CID']+parseUrl.search;
     var buttonPrev = param.dom.btnPrev;
     var buttonNext = param.dom.btnNext;

     buttonPrev.html('<a href="'+prevUrl+'"><span>prev</span></a>');
     buttonNext.html('<a href="'+nextUrl+'"><span>next</span></a>');
     }
     });
     */

    /** [Photo Auto List AutoAlign] Controller */
    window.YNA_CORE('PhotoGrid', {
        render : function(PhotoBox){
            PhotoBox.css('opacity', 0);
            var GRD = window.window.YNA_CORE('AutoAlign', '.photo-grid');
            if( !GRD.checkDevice ){ return false; }
            if( GRD.checkDevice()['currentDevice'] === 'mobile'){
                GRD.init({
                    defaultNumber : 4,
                    isAnimation: false,
                    landscapeResize : true,
                    mobileLandscape : { portrait  : 2, landscape : 3 },
                    tabletLandscape : { portrait  : 3, landscape : 4 },
                    lazyImage : true,
                    lazyLoad : true,
                    lazyLoadSpeed : 700,
                    lazyComplete : function(){
                        PhotoBox.fadeTo("medium", 1);
                        GRD.changeAutoPos(2);
                    }
                });
            } else {
                GRD.init({
                    gapX : 20,
                    gapY : 20,
                    defaultNumber : 4,
                    isAnimation: false,
                    landscapeResize : false,
                    lazyImage : true,
                    lazyLoad : true,
                    lazyLoadSpeed : 700,
                    lazyComplete : function(){
                        PhotoBox.fadeTo("medium", 1);
                        GRD.changeAutoPos(4);
                    }
                });
            }
        }
    });

    /** [TopNews] Week/Month Controller */
    TopNews.init({
        domainParse     : (setImgDomain)?setImgDomain:null,
        className       : '.yna-date-list',
        dateChangeGroup : '.left-con',
        weekMonthGroup  : '.right-con',
        dateListGroup   : '.tn-list-all',
        prevRegDate     : '',
        moreButtonCheck : function(lang){
            var moreType = {
                'EN' : '<div class="btns-con"><button class="btn-more04"><span>LOAD MORE</span></button></div>',
                'CK' : '<div class="btns-con"><button class="btn-more04"><span>查看更多内容</span></button></div>',
                'JP' : '<div class="btns-con"><button class="btn-more04"><span>もっと読む</span></button></div>',
                'AR' : '<div class="btns-con"><button class="btn-more04"><span>تحميل مزيد</span></button></div>',
                'SP' : '<div class="btns-con"><button class="btn-more04"><span>MAS</span></button></div>',
                'FR' : '<div class="btns-con"><button class="btn-more04"><span>VOIR PLUS D\'ARTICLES</span></button></div>'
            };
            return moreType[lang];
        },
        snsTypeCheck : function(lang, snsString, snsTitle){
            var snsType = {
                'ALL' :
                '<button type="button" class="share-btn fb" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Facebook</span></button>' +
                '<button type="button" class="share-btn tw" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Twitter</span></button>' +
                '<button type="button" class="share-btn pin" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Pinterest</span></button>' +
                '<button type="button" class="share-btn lin" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Linked in</span></button>' +
                '<button type="button" class="share-btn tum" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Tumblr</span></button>' +
                '<button type="button" class="share-btn red" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Reddit</span></button>' +
                '<button type="button" class="share-btn fbm" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Facebook Messenger</span></button>',
                'CK' :
                '<button type="button" class="share-btn wei" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Weibo</span></button>' +
                '<button type="button" class="share-btn qq" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>QQ空間</span></button>' +
                '<button type="button" class="share-btn wec" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Wechat</span></button>' +
                '<button type="button" class="share-btn ren" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Renren</span></button>' +
                '<button type="button" class="share-btn fb" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Facebook</span></button>' +
                '<button type="button" class="share-btn tw" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Twitter</span></button>',
                'JP' :
                '<button type="button" class="share-btn fb" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Facebook</span></button>' +
                '<button type="button" class="share-btn tw" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Twitter</span></button>' +
                '<button type="button" class="share-btn hb" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Hatena</span></button>' +
                '<button type="button" class="share-btn fbm" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Facebook Messenger</span></button>',
                'AR' :
                '<button type="button" class="share-btn fb" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Facebook</span></button>' +
                '<button type="button" class="share-btn tw" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Twitter</span></button>' +
                '<button type="button" class="share-btn fbm" data-snslink="'+snsString+'" data-title="'+snsTitle+'"><span>Facebook Messenger</span></button>'
            };
            if( lang === 'EN' || lang === 'SP' || lang === 'FR'){
                return snsType['ALL'];
            } else {
                return snsType[lang];
            }
        },
        stringMore : function(){
            var result = '';
            var langType = (window.LANG_TYPE)?window.LANG_TYPE.toUpperCase():'EN';
            result = this.moreButtonCheck(langType);
            return result;
        },
        stringSns : function(snsString, snsTitle, likeId){
            var result = '';
            var langType = (window.LANG_TYPE)?window.LANG_TYPE.toUpperCase():'EN';
            var snsDom = this.snsTypeCheck(langType, snsString, snsTitle);
            /** 공유 버튼 */
            result += '<dl class="option-type01">';
            result += '<dt class="blind">Article View Option</dt>';
            result += '<dd>';
            result += '<button class="btn-share"><span>SHARE</span></button>';
            if( like.check(likeId) ) result += '<button class="btn-like btnlike-on" data-likeid="'+likeId+'"><span>LIKE</span></button>';
            else result += '<button class="btn-like" data-likeid="'+likeId+'"><span>LIKE</span></button>';
            result += '</dd>';
            result += '</dl>' +
                /** 공유 박스 */
                '<div class="pop-share jssocials">' +
                '<div class="sns-share jssocials-shares">'+snsDom+'</div>' +
                /** 공유 박스 */
                '<button type="button" class="btn-sns-close"><span>close</span></button>' +
                '<span class="bg"></span>' +
                '</div>';
            return result;
        },
        onChangeDateRange : function(dom, state){
            this.request(dom, state);
        },
        onLoad : function(res, err){
            if( !err ){
                this.request(res.dom, res.state);
            } else {
                //console.log('실행 실패!');
            }
        },
        onShareSns : function(param){
            //console.log('share', param)
        },
        onLike : function(param){
            //console.log('like', param)
        },
        request : function(dom, state){
            var module = this;
            var sDate = state.sDate.fullString;
            var eDate = state.eDate.fullString;
            var page = state.page;
            var count = state.count;
            var ids = TOPNEWS_ID;
            var url = APS+'hedit.date?id='+ids+'&body=Y&total=Y&from='+sDate+'&to='+eDate+'&page='+page+'&count='+count;
            url = encodeURI(url);
            $.ajax({
                url : url,
                method : 'GET',
                async: false,
                cache : true,
                success : function(json){
                    var data = json.DATA;
                    if( json && data != '' ){
                        state.total = json.TOTAL.DATA[0].TOTAL;
                        module.render(dom, state, data);
                    } else {
                        $('.tn-list-all').html('');
                    }
                },
                fail : function(){
                    module.onLoad('ajaxError', true);
                }
            });
        },
        getBeforeDayDate : function(num, date){
            var tempDate = (date)?new Date(date / 10000, (date % 10000 / 100) - 1, date % 100):new Date();
            var dayOfMonth = tempDate.getDate();
            tempDate.setDate(dayOfMonth - num);
            var year = parseInt(tempDate.getFullYear());
            var month = parseInt(tempDate.getMonth());
            var monthNum = month+1;
            var monthStr = ('0'+monthNum);
            monthStr = monthStr.substr(monthStr.length-2, 2);
            var day = parseInt(tempDate.getDate());
            var dayStr = ('0'+day);
            dayStr = dayStr.substr(dayStr.length-2, 2);
            return year+monthStr+dayStr;
        },
        render : function(dom, state, data){
            var lang = LANG_TYPE;
            var module = this;
            var target = $('.tn-list-all');
            var div = '<div class="tn-list-unit ynaListGroup"></div>';
            var moreButton = module.stringMore();
            var location = window.location;
            var protocol = window.location.protocol;
            var prevRegDate = module.prevRegDate;
            if(prevRegDate == ''){
                if(state.dateType == 'week'){
                    prevRegDate = state.sDate.fullString;
                } else {
                    prevRegDate = state.wSDate.fullString;
                }
            }

            if( state.page == 1 ){
            	target.html('');
            }
            var div_index = $('.tn-list-unit').length-1;
            var getBeforeDayDate = function(num, date){
                var d = (date)?new Date(date):new Date();
                var dayOfMonth = d.getDate();
                var monthOfYear = d.getMonth();
                d.setDate(dayOfMonth + num);
                return TopNews.getDate(d);
            };
            $.each(data, function(idx, val){
                var li = '';
                var regDate = val['REGIST_DATETIME'];

                if(state.dateType == 'week'){
                    regDate = regDate.slice(0,8);
	                if( prevRegDate != regDate ){
	                    makeTopNewsLayout(target, div, state, lang, regDate);
	                    prevRegDate = regDate;
                        module.prevRegDate = regDate;
	                    div_index += 1;
	                }
                } else {
                    regDate = regDate.slice(0,8);
                    if( regDate < prevRegDate ){
                        while( regDate < state.wSDate.fullString ){
                            state.wEDate = getBeforeDayDate(-1,state.wSDate.date);
                            state.wSDate = getBeforeDayDate(-6,state.wEDate.date);
                        }
                        makeTopNewsLayout(target, div, state, lang);
                        prevRegDate = state.wSDate.fullString;
                        module.prevRegDate = prevRegDate;
                        div_index += 1;
                    }else{
                        if( state.page == 1 && idx == 0 ) {
                            makeTopNewsLayout(target, div, state, lang);
                            prevRegDate = state.wSDate.fullString;
                            module.prevRegDate = prevRegDate;
                            div_index += 1;
                        }
                    }
                }

                var _imageUrl = module.domainParse(val['IMG'] || '') || '';
                var _sectionName = val['SECTION_NAME'];
                var _dateTime = val['DATETIME'];
                _dateTime = changeDateFormat.setTopnewsFormat(val['DATETIME'], LANG_TYPE);
                var _getDateInfo = TopNews.getDate(_dateTime);
                var _articleTitle = val['TITLE'];
                var _articleBody = val['BODY'];
                var _articleUrl = val['URL'];
                if( val['KEYWORD'] != null && val['KEYWORD'] != '' ){
                    var _articleKeyword = val['KEYWORD'].split(';')[0];
                    var _articleKeywordUrl = '/search/index?query='+encodeURI(this)+'&lang='+lang.toUpperCase();
                }
                var _snsUrl = location.origin+_articleUrl;
                var likeId = UI.getArticleIdFromString(_snsUrl) || '';
                var snsString = module.stringSns(_snsUrl, _articleTitle, likeId);
                if( _imageUrl ){
                    _imageUrl = protocol+_imageUrl+val['IMG'];
                    _imageUrl = _imageUrl.replace('_T', '_P2');
                } else {
                    _imageUrl = 'data:image/gif;base64,R0lGODlhAQABAIAAAAUEBAAAACwAAAAAAQABAAACAkQBADs='; //임시
                }

                li += '<li data-likeid="'+likeId+'" data-snslink="'+_snsUrl+'">' +
                    '<article data-url="'+_snsUrl+'">' +
                    '<figure class="img-cover"><a href="'+_snsUrl+'"><img src="'+_imageUrl+'" alt=""></a></figure>'+snsString +
                    '<div class="txt-con">' +
                    '<span class="tag">'+ (val['KEYWORD'] != null  && val['KEYWORD'] != '' ? '<a href="'+_articleKeywordUrl+'">'+'#'+_articleKeyword+'</a>' : '')+'</span>'+
                    '<h2 class="tit"><a href="'+_snsUrl+'">'+_articleTitle+'</a></h2>' +
                    '<span class="date"><a href="'+_snsUrl+'">'+_sectionName+'</a> '+_dateTime +'</span>' +
                    '</div>' +
                    '</article>' +
                    '</li>';

                $('.tn-list-unit ul').eq(div_index).append(li);
            });

            if( $('.tn-list-all li').length < state.total ) {
                target.append(moreButton);
            } else {
                $('.btns-con').hide();
            }

            $('.btns-con').off().on('click', function(){
                $(this).remove();
                state.page += 1;
                module.request(dom, state);
            });

            // 이미지 크롭함수(global/common.js)
            imgCrop();
            // 좋아요 토글 적용
            like.setEvent();
        }
    });

    function makeTopNewsLayout(target, div, state, lang, prevRegDate){
        var tmp_string = "";
        if(state.dateType == 'week'){
        	var tmp_date = changeDateFormat.stringToDate(prevRegDate);
            if( lang == 'en' )	tmp_string = tmp_date.Format2('MMM. dd',lang);
            else if( lang == 'ck' ) tmp_string = tmp_date.Format2('MM月dd日',lang);
            else if( lang == 'jp' ) tmp_string = tmp_date.Format2('MM.dd', lang);
            else if( lang == 'ar' ) tmp_string = tmp_date.Format2('MM.dd', lang);
            else if( lang == 'sp' ) tmp_string = tmp_date.Format2('d1 MMM', lang).toLowerCase();
            else tmp_string = tmp_date.Format2('dd.MM', lang);
        } else {
            var tmp_sDate = state.wSDate.date;
            var tmp_eDate = state.wEDate.date;

            if( lang == 'en' ){
            	var sMonthStr = tmp_sDate.Format2('MMM',lang);
            	var eMonthStr = tmp_eDate.Format2('MMM',lang);
            	tmp_string  = tmp_sDate.Format2('MMM. dd-',lang);
            	if(sMonthStr != eMonthStr){
            		tmp_string += tmp_eDate.Format2('MMM.',lang);
            	}
            	tmp_string += tmp_eDate.Format2('dd');

            } else if( lang == 'ck' ) {
                var sMonthStr = tmp_sDate.Format2('MMMM',lang);
                var eMonthStr = tmp_eDate.Format2('MMMM',lang);
                tmp_string  = tmp_sDate.Format2('MM月dd日-',lang);
                if(sMonthStr != eMonthStr){
                    tmp_string += tmp_eDate.Format2('MMMM',lang);
                }
                tmp_string += tmp_eDate.Format2('dd日');
            }  else if( lang == 'jp' ) {
                var sMonthStr = tmp_sDate.Format2('MM');
                var eMonthStr = tmp_eDate.Format2('MM');
                tmp_string  = tmp_sDate.Format2('MM.dd-');
                if(sMonthStr != eMonthStr){
                    tmp_string += tmp_eDate.Format2('MM.');
                }
                tmp_string += tmp_eDate.Format2('dd');
            } else if( lang == 'ar' ) {
                var sMonthStr = tmp_sDate.Format2('MM');
                var eMonthStr = tmp_eDate.Format2('MM');
                tmp_string  = tmp_sDate.Format2('MM.dd-');
                if(sMonthStr != eMonthStr){
                    tmp_string += tmp_eDate.Format2('MM.');
                }
                tmp_string += tmp_eDate.Format2('dd');
            } else if( lang == 'sp' ) {
                var sMonthStr = tmp_sDate.Format2('MM');
                var eMonthStr = tmp_eDate.Format2('MM');
                if(sMonthStr != eMonthStr){
                    tmp_string = tmp_sDate.Format2('d1 de MMM', lang).toLowerCase()+' - ';
                }else{
                    tmp_string = tmp_sDate.Format2('d1 - ', lang);
                }
                tmp_string += tmp_eDate.Format2('d1 de MMM', lang).toLowerCase();
            } else {
                var sMonthStr = tmp_sDate.Format2('MM');
                var eMonthStr = tmp_eDate.Format2('MM');
                if(sMonthStr != eMonthStr){
                    tmp_string = tmp_sDate.Format2('dd.MM-');
                }else{
                    tmp_string = tmp_sDate.Format2('dd-');
                }
            	tmp_string += tmp_eDate.Format2('dd.MM');
            }
        }
        var clone = $(div).clone().html('<span class="i-date">'+tmp_string+'</span><div class="smain-photo-type03"><ul class="listUl"></ul></div>');
        target.append(clone);
    }

    /** 메인 자동 그리드 모듈 컨트롤 */
    $(function(){
        if( location.href.match('m-') && $('.grid.ssc-autopos-wrap').length > 0 ){
        	// 앱에서 제거돼야할 컴포넌트가 있으면 내용 제거(19.02.25 추가)
        	if( /YonhapnewsApp/i.test(navigator.userAgent) && $('.grid.ssc-autopos-wrap .app-hide').length ) {
        		$('.app-hide').html('');
        	}
            userModule = window.window.YNA_CORE('AutoAlign', '.grid');
            var arr2 = {
                'EN' : [[1, 11, 8, 14, 16, 20, 21, 3, 2, 6, 7, 9, 5], [15, 17, 22, 4, 10, 12, 13, 18, 19, 23]],
                'CK' : [[1, 11, 10, 14, 7, 18, 20, 3, 2, 6, 8, 5], [19, 9, 17, 4, 12, 13, 15, 16, 21]],
                'JP' : [[1, 10, 6, 13, 15, 19, 3, 2, 8, 12, 5], [14, 18, 7, 4, 9, 11, 16, 17, 20]],
                'AR' : [[4, 12, 11, 16, 13, 20, 3, 6, 8, 9, 5], [18, 19, 1, 10, 7, 14, 17]],
                'SP' : [[1, 8, 6, 11, 16, 3, 2, 7, 5], [12, 17, 4, 9, 10, 15, 14, 13, 18]],
                'FR' : [[1, 9, 6, 12, 17, 3, 2, 7, 8, 5], [15, 16, 18, 4, 10, 11, 13, 14, 19]]
            };
            var arr3 = {
                'EN' : [[1, 11, 8, 14, 16, 20, 21, 5], [3, 2, 6, 7, 9, 15, 17, 22], [4, 10, 12, 13, 18, 19, 23]],
                'CK' : [[1, 11, 10, 14, 7, 18, 20, 5], [3, 2, 6, 8, 19, 9, 17], [4, 12, 13, 15, 16, 21]],
                'JP' : [[1, 10, 6, 13, 15, 19, 5], [3, 2, 8, 12, 14, 18, 7], [4, 9, 11, 16, 17, 20]],
                'AR' : [[4, 12, 11, 16, 13, 20, 5], [3, 6, 8, 9, 18, 19], [1, 10, 7, 14, 17]],
                'SP' : [[1, 8, 6, 11, 16, 5], [3, 2, 7, 12, 17], [4, 9, 10, 15, 14, 13, 18]],
                'FR' : [[1, 9, 6, 12, 17, 5], [3, 2, 7, 8, 15, 16, 18], [4, 10, 11, 13, 14, 19]]
            };
            
            var width_size = window.innerWidth;
            if( width_size > 767 ) {
            	userModule.init({
                    defaultNumber : 1,
                    isAnimation: false,
                    landscapeResize : true,
                    orderChangeByGroup : true,
                    orderChangeUserSetting : {
                        group_2: arr2[LANG_TYPE.toUpperCase()],
                        group_3: arr3[LANG_TYPE.toUpperCase()]  //3그룹 순서
                    }
                });
            	
            	if( width_size > 1024 ) {
            		userModule.changeAutoPos(3);
            	} else {
            		userModule.changeAutoPos(2);
            	}
            }

            $(window).resize(function(){
            	if( width_size != window.innerWidth ){
            		if( window.innerWidth < 768 ) window.location.reload();
                    else if( window.innerWidth < 1025 ) {
                    	if( width_size < 768) window.location.reload();
                    	else userModule.changeAutoPos(2);
                    }
                    else if( window.innerWidth >= 1025 ) {
                    	if( width_size < 768 ) window.location.reload();
                    	else userModule.changeAutoPos(3);
                    }
            	}
            });
        }
        /*
        if( UI.checkDevice() === 'desktop' ){}
        else {}
        */
    });

    /** 페스티벌 페이지 처리 : 2018.07.25 급 요청 - 객체모듈로 만들지 않고 기본 정의하겠음 */
    (function festivalEffect(){
        var OPT = {
            active         : true,
            isFestivalPage : $('.yna-festivals-dom'),
            isSectionValue : UI.isSectionValue(),
            isFestivalZone : $('.festival-calendar'),
            monBtn         : $('.right-con li.mon'),
            ImageType      : 'P1',
            QueryData      : UI.getUrlQuery()['query'],
            UrlData        : UI.getUrlQuery()['urlData'],
            LANG           : (window.LANG_TYPE)?window.LANG_TYPE.toUpperCase():'KR'
        };
        /** 일단.... 상세보기와.. index 페이지를 체크했을 경우.... */
        if( OPT.isFestivalZone.length > 0 || OPT.isSectionValue === 'festivals/all' ){
            var crtCid = OPT.UrlData['path'].replace('/view/', '');
            var fadeToContent = function(){ OPT.isFestivalZone.fadeTo('slow', 1); };
            var getInitialSlickNum = function(){
                var linkCid = OPT.QueryData['linkcid'];
                var slickBox = $('.pto-view-con');
                $.each(slickBox, function(num){
                    if( $(this).data('linkcid') == linkCid ) Window.INIT_SLICK_NUM = num;   // slick  실행부는 global/js/common.js 안에 있음.
                });
            };
            var renderContentDom = function(param){
                var linkCid = OPT.QueryData['linkcid'];
                if( linkCid == undefined ) linkCid = param[0]['LINK_CID'];
                var slickBox = $('.fc-slick');
                var bottomListBox = $('.list-type-photo ul');
                var slickCrtNum = 0;
                var slickHtml = '';
                var slickItems = [];
                var bottomHtml = '';

                $.each(param, function(num){
                    var crtLinkCid = this['LINK_CID'];
                    var imgUrl = this['IMG'];
                    var findImgStr = imgUrl.split('.');
                    var splitStr = findImgStr[0].split('_');
                    var lastStr = splitStr[splitStr.length-1];
                    var date = changeDateFormat.setArticleFormat(this['DATETIME'], LANG_TYPE);
                    var body_arr = this['LINK_CAPTION'].split('\n');
                    var body = '';
                    $.each(body_arr, function(idx, b){
                        body += '<p>'+b+'</p>';
                    });
                    splitStr[splitStr.length-1] = (lastStr === 'T')?OPT.ImageType:'P2';
                    findImgStr[0] = splitStr.join('_');
                    findImgStr = findImgStr.join('.');
                    var slickItem = '<div class="pto-view-con" data-linkcid="'+crtLinkCid+'"><figure><img src="'+ImgDomain+findImgStr+'" alt="'+this['TITLE']+'" title="'+this['TITLE']+'" /></figure><div class="txt-desc"><h2 class="tit">'+this['TITLE']+'</h2>'+body+'</div></div>';
                    var bottomItem = '<li><article><figure class="img-con img-cover"><a><img src="'+ImgDomain+findImgStr+'" alt="'+this['TITLE']+'" title="'+this['TITLE']+'"></a></figure><div class="txt-con"><a><h2 class="tit">'+this['TITLE']+'</h2></a><span class="date">'+date+'</span></div></article></li>';
                    slickItems.push(slickItem);
                    bottomHtml += bottomItem;
                    if( crtLinkCid === linkCid ){ slickCrtNum = num; }
                });
                slickHtml = slickItems.join('');
                slickBox.html(slickHtml);
                bottomListBox.html(bottomHtml);
                fadeToContent();
            };
            var callAjaxItems = function(cid){
                var url = encodeURI(ISSUE_API+cid+'&all=Y');
                $.ajax({
                    url : url,
                    method : 'GET',
                    async: false,
                    cache : true,
                    success : function(data){
                        var param = data['DATA'];
                        if( param && param.length > 0 ){
                            renderContentDom(param);
                        }
                    },
                    fail : function(){
                        console.log('error load festival data!');
                    }
                });
            };
            var renderTopArrow = function(param){
                var datePeriodTitle = $('.date-period');
                var paramLength = param.length;
                var index = -1;
                if( crtCid === '/festivals/index' ){crtCid = param[0]['CID'];}
                $.each(param, function(num){
                    var thisCid = this['CID'];
                    if( thisCid === crtCid ){
                        datePeriodTitle.text(this['TITLE']);
                        var $btn_dateNext = $('.date-next');
                        var $btn_datePrev = $('.date-prev');

                        if( num !== paramLength-1 ){
                            var cid_left = param[num+1]['CID'];
                            var link_cid_left = param[num+1]['LINK_PHOTO_ID'];
                            var link_left = OPT.UrlData['origin']+'/view/'+cid_left+'?section=festivals/all&linkcid='+link_cid_left;
                        }else{ $btn_datePrev.remove(); }
                        if( num > 0 ){
                            var cid_right = param[num-1]['CID'];
                            var link_cid_right = param[num-1]['LINK_PHOTO_ID'];
                            var link_right = OPT.UrlData['origin']+'/view/'+cid_right+'?section=festivals/all&linkcid='+link_cid_right;
                        }else{ $btn_dateNext.remove(); }
                        /** Event Bind*/
                        $btn_dateNext.on('click', function(){window.location.href = link_right;return false;});
                        $btn_datePrev.on('click', function(){window.location.href = link_left;return false;});
                        index = num;
                        return false;
                    }
                });
                if( OPT.isSectionValue === 'festivals/index' ) {
                    if( index == -1 ) callAjaxItems(crtCid);
                    else callAjaxItems(param[index]['CID']);
                }
                getInitialSlickNum();
            };
            (function(init){
                if( !init ){
                    fadeToContent();
                    return false;
                }
                var url = encodeURI(ISSUEEX_API+OPT.LANG+'&div=I060100016&limit=12&rcount=-1');
                $.ajax({
                    url : url,
                    method : 'GET',
                    async: false,
                    cache : true,
                    success : function(data){
                        var param = data['DATA'];
                        if( param && param.length > 0 ){
                            if(OPT.isFestivalPage.length != 0){
                                //var firstBox = $('.view-box').find('.list-type-photo').first().find('li a');
                                //if( !firstBox ){OPT.monBtn.remove();}
                                //else {OPT.monBtn.find('a').attr('href', firstBox.attr('href'));}
                                fadeToContent();
                            } else {
                                //OPT.monBtn.find('a').attr('href', window.location.href);
                                renderTopArrow(param);
                            }
                        }
                    },
                    fail : function(){
                        console.log('error load festival data!');
                    }
                });
                if( OPT.isFestivalZone.length > 0 && OPT.UrlData['path'] !== '/festivals/all' ) {
                    var linkCid = OPT.QueryData['linkcid'];
                    var bottomList = $('.list-type-photo li');
                    var index = -1;
                    if( linkCid != undefined ) {
                        $.each($('.list-type-photo li'), function(idx){
                            if( linkCid == $(this).data('linkcid') ) index = idx;
                        });
                        if( index == -1 ) bottomList.eq(0).addClass('on');
                        else bottomList.eq(index).addClass('on');
                    } else {
                        bottomList.eq(0).addClass('on');
                    }
                }
            }(OPT.active));
        }
    }());

    /** TODO : 테스트용으로 만든 모듈, 브라우저 및 보안상의 이유로 구동이 안됨 */
    if( IsTestAllow ){
        /** [GeoLocation] controller */
        window.YNA_CORE('GeoLocation', {
            userLocation : function(param){
                console.log(param)
            }
        });
        /** [Slider] Controller */
        window.YNA_CORE('Slider', {
            sliderClass : '.yna-photo-slider',
            sliderOption : {
                slidesPerView: 1,
                loop: true,
                grabCursor: true,
                pagination: {
                    el: '.swiper-pagination',
                    clickable: true
                },
                navigation: {
                    nextEl: '.swiper-button-next',
                    prevEl: '.swiper-button-prev'
                }
            },
            request : function(module){
                /*
                 var ajaxUrl = 'ajaxTest/photo.json';
                 $.ajax({
                 url : ajaxUrl,
                 method : 'GET',
                 async: false,
                 cache : true,
                 success : function(data){
                 var result = data['DATA'];
                 module.initSlider(result);
                 },
                 fail : function(){console.log('photo-ajax-error!')}
                 });
                 */
            },
            render : function(){

            }
        });
        /** TODO : 알람 모듈은 브라우저 지원에 문제가 있음 */
        /** notify controller */
        var NOTIFY = YNA_CORE('Notify',{
            title     : "알람테스트 시작",
            content   : "지금부터 알람테스트를 시작합니다!!!",
            autoClose : true,
            closeTime : 5
        });
        NOTIFY.message({
            title : "연합뉴스 알람테스트~!",
            content : "문 대통령은 '2015년 한일 양국 정부 간 위안부 협상은 절차적으로나 " +
            "내용으로나 중대한 흠결이 있었음이 확인됐다'며 '유감스럽지만 피해갈 수는 없다'고 했다."
        });
    }
});
/** [YNA Control] Module End ------------------------------------------------------------ */

/** window.CP_Callback : addCallback ---------------------------------------------------- */
function addCallback(cb){
    if (window.CP_Callback) {
        var cp = window.CP_Callback;
        if (typeof cp == 'object' && isNaN(cp.length) === false) {
            cp.push(cb);
        } else {
            var list = [];
            list.push(cp);
            list.push(cb);
            window.CP_Callback = list;
        }
    } else {
        window.CP_Callback = cb;
    }
}
/** window.CP_Callback : addCallback END ------------------------------------------------ */

/** setImgDomain ------------------------------------------------------------------------ */
function imgCrop(){
    // 썸네일 이미지 크롭
    $('.img-cover').imgLiquid({fill:true, verticalAlign:'top'});
}
// dev-img[0-9]에서 숫자 정하는 hash
function hashmod(str,mod) {
    var h = 0;
    for (var i=0; i<str.length; i++)
    {
        h = 0xffffffff & (31 * h + str.charCodeAt(i));
    }
    return Math.abs(mod ? h % mod : h % 10);
}
function setImgDomain(img_src){
    var result = '';
    if (img_src){
        var dot = IMG_DOMAIN.indexOf('.');
        result = IMG_DOMAIN.slice(0, dot) + hashmod(img_src, 10).toString() + IMG_DOMAIN.slice(dot);
    }
    return result;
}
/** setImgDomain : END ------------------------------------------------------------------- */