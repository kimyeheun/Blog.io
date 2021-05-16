/** -------------------------------------------------------------------------------------- /
 * ModuleName  : mediaCompiler Core Module.
 * Developer   : SeoulSystem.corp
 * DevLanguage : Javascript + jQuery
 * BuildDate   : 2018.01.26
 * Information : 연합뉴스 서비스 페이지의 콤퍼넌트 커스텀 태그를 활성화는 모듈을 정의한다.
 *               1) YHML 소재[영상,사진,화보,SNS,하이퍼링크]를 정규 HTML 문법으로 변환
 *               2) 더보기 동작 실행
 *               3) 본문 내 기능[본문 내 구글광고 위치 변경, LANG_TYPE 이 다른 기사일 시 redirect]
 *               사용문법은 Javascript 를 사용하며 개발기간 단축을 위해 jQuery 를 사용한다.
 * --------------------------------------------------------------------------------------- */


(function(window, $){
    'use strict';

    var LANG = (window.LANG_TYPE)?window.LANG_TYPE.toUpperCase():'EN';
    var LANG_ARRAY = ['KR', 'EN', 'CK', 'JP', 'AR', 'SP', 'FR'];
    var LANG_NUM = LANG_ARRAY.indexOf(LANG || 'KR');

    /** Check Window Config */
    if( !$ ){ alert('Please! jQuery Module Check'); return; }
    if( window.YNA_SERVICE ){
        /** 본문 보기에서 스크롤 발생시에 중앙 컨텐츠와 우측 컨튼츠의 높이 계산 오류를 수정한다. */
        window.YNA_SERVICE['RESIZE'] = function(){
            /** TODO: 광고 컴파일링 삽입후 컨텐츠 높이 계산을 다시 해서 include
             * api 호출 방식의 컨텐츠 삽입 방식으로 전환되었기 때문에
             * 초기 태그의 동적 높이 계산을 통한 스크롤 이벤트 처리에 문제가 있어보임 (common2018.js)
             * */
            setTimeout(function(){
                var gap = 0;
                var checkContentBody = $('.scroll-start01');
                var checkContentSide = $('.scroll-start02');
                var bodyInnerHeight = parseInt(checkContentBody.children().height());
                var sideInnerHeight = parseInt(checkContentSide.children().height());
                checkContentBody.css({ height: bodyInnerHeight+gap });
                checkContentSide.css({ height: sideInnerHeight+gap});
            }, 1000)
        };
        window.YNA_SERVICE['MODULE'] = {};
    }
    if( !window.jsonpCallback ){ window.jsonpCallback = function(data){/**/}} //사용안함 지워도 됨
    /** CDN URL Common */
    window.YNA_SERVICE['MODULE']['ynaCdnUrl'] = 'http://cdnvod.yonhapnews.co.kr/yonhapnewsvod/meps/';

    /** [default setting] Module*/
    var MultiGroupCompiler = null;
    var PhotoGroupCompiler = null;
    var VideoGroupCompiler = null;
    var PhotoIPTCompiler = null;
    var SocialGroupCompiler = null;
    var HyperGroupCompiler = null;
    var AdvertisementCompiler = null;
    var ComponentContentCompiler = null;
    var MoreContentModule = null;
    var LANG = (window.LANG_TYPE)?window.LANG_TYPE.toUpperCase():'KR';

    /** [MediaGroupCompiler] Module Start */
    MultiGroupCompiler = function(props){
        this.group = {};
        this.props = {
            isModuleActive: true,
            customTagName : 'yna',
            mediaUrl      : window.YNA_SERVICE['MODULE']['ynaCdnUrl'],
            groupClass    : '.yna_group_slide',
            videoDataName : 'yhnVideoType',
            videoCodeName : 'mpic',
            videoTitle    : 'Videos',
            imageCodeName : 'image',
            imageTitle    : 'Photos',
            isFadeIn      : false,
            parseVideoStr : ['T', 'T2', 'P1', 'P2', 'P3', 'P4'],
            prevBtnClass  : '.btn-prev02',
            nextBtnClass  : '.btn-next02'
        };
        this.langDef = {
            show : ['', 'show caption', '展开', 'キャプションを読む', 'مشاهدة التعليق', 'Ver el texto', 'Voir la légende'],
            hide :['', 'hide caption', '收起', 'キャプションを隠す', 'إخفاء التعليق', 'Ocultar el texto', 'Cacher la legende']
        };
        this.init(props);
        return this.group;
    };
    MultiGroupCompiler.prototype = {
        onStartLayout : function(mBox){
            var module = this;
            $.each(mBox, function(){
                var target = $(this);
                var items = target.find('.slide-article .yhnSlideItem');
                var firstItem = items.first();
                var lastItem = items.last();
                if( firstItem && lastItem ){
                    firstItem.find(module.props.prevBtnClass).remove();
                    lastItem.find(module.props.nextBtnClass).remove();
                }
            });
        },
        onStartCaption : function(mBox){
            $.each(mBox, function(){
                var target = $(this);
                var slideArticle = target.find('.slide-article .yhnSlideItem')[0];
                var capZone = target.find('.caption-zone');
                var targetCaptionData = $(slideArticle).find('.yhnCaption');
                capZone.find('span.title').text(targetCaptionData.attr('data-title')).end()
                    .find('p.desc').text(targetCaptionData.text());
                //.find('span.type').text(targetCaptionData.attr('data-type')).end()
            });
        },
        onVideoJsStart : function(){
            var video = $('body').find('video[data-name="'+this.props.videoDataName+'"]');
            if( window.videojs ){
                $.each(video, function(){ videojs(this.id); });
            } else {
                //console.log('[error] file Error - Check Please videoJs file!!');
            }
        },
        createSliderDom : function(key, array){
            var module = this;
            var dom = this.getSlideGroupDom(array);
            var mainSlide = [];
            var thumbSlide = [];
            dom.attr('data-group', key);
            $.each(array, function(num, node){
                var item = $(node);
                mainSlide.push(module.getSlideMainContentDom(item));
                thumbSlide.push(module.getSlideThumbContentDom(item));
            });
            dom.find('.slide-article').append($(mainSlide));
            dom.find('.thumb-imgs').append($(thumbSlide));
            try {
                return dom;
            } finally {
                mainSlide = null;
                thumbSlide = null;
            }
        },
        getSlideGroupDom : function(array){
            var totalLength = array.length;
            var dom = $('<div class="comp-box slider-group pic-zone" data-total="'+totalLength+'"></div>');
            var inner = $('<div class="inner"></div>');
            var slideArticle = '<div class="slide-article"></div>';
            var capZone = '<div class="caption-zone"><!--<strong><span class="type"></span>&nbsp;:&nbsp;</strong><span class="title"></span>--><p class="desc"></p></div>';
            var button = '<button type="button" class="btn-caption btn"><span class="txt-show">'+this.langDef.show[LANG_NUM]+'</span><span class="txt-hide">'+this.langDef.hide[LANG_NUM]+'</span></button>';
            /** 썸네일 슬라이드 영역 */
            var thumbBox = $('<div class="thumb-box"></div>');
            var thumbControl = '<div class="control-box"><div class="controller"><button type="button" class="btn-prev">previous</button><button type="button" class="btn-next">next</button></div><span class="num">' +
                '<strong>1</strong><span>&nbsp;of&nbsp;</span><span> '+totalLength+'</span></span></div>';
            if( LANG_TYPE != 'ar' )
                var thumbSlide = '<div class="thumb-list-wrap"><div class="thumb-imgs slider-nav"></div></div>';
            else
                var thumbSlide = '<div class="thumb-list-wrap" dir="rtl"><div class="thumb-imgs slider-nav"></div></div>';
            thumbBox.append(thumbControl, thumbSlide);
            inner.append(slideArticle, capZone, button, thumbBox);
            dom.append(inner);
            try {
                return dom;
            } finally {
                dom = null;
                inner = null;
                slideArticle = null;
                capZone = null;
                button = null;
                thumbBox = null;
                thumbControl = null;
                thumbSlide = null;
            }
        },
        getSlideMainVideoDom : function(node){
            var imgViewPath = node.attr('data-viewpath');
            var imgThumbPath = node.attr('data-thumnailpath');
            var imgTitle = node.attr('data-title');
            var imgAlt = node.attr('data-alt');
            if( $.trim(imgTitle).length < 1 ) imgTitle = $('meta[name="title"]').attr('content');
            var imgDesc = node.attr('data-caption');
            var videoSrcArray = imgThumbPath.split('.');
            var refid = node.attr('data-refid');
            var refDate = refid.substr(3,6);
            var videoUrl = this.props.mediaUrl+refDate+'/'+refid+'_700M1.mp4';
            /** TODO : 잠시보류 */
            var todoFnc = function(){
                /* 이미지 확장자 변환 */
                videoSrcArray[videoSrcArray.length-1] = 'mp4';
                /* 특수 string 변환 */
                var lastCheck = (videoSrcArray[videoSrcArray.length-2]).split('_');
                if( lastCheck.length > 1 ){
                    if( this.props.parseVideoStr.indexOf(lastCheck[lastCheck.length-1]) !== -1 ){ lastCheck[lastCheck.length-1] = 'm' }
                    lastCheck = lastCheck.join('_');
                    videoSrcArray[videoSrcArray.length-2] = lastCheck;
                }
                videoSrcArray = videoSrcArray.join(".");
            };
            var now = $.now();
            //TODO : var videoId = 'yhnVdo_'+now;
            var videoId = 'todayVideoPlay';
            var videoWrap = $('<div class="video"></div>');
            var videoButton = '<button type="button" class="btn-prev02">previous</button>' +
                '<button type="button" class="btn-next02">next</button>';
            var video = $('<video id="'+videoId+'" data-name="'+this.props.videoDataName+'" class="video-js vjs-default-skin vid-skin02" controls preload="auto" width="750" height="420" poster="'+imgViewPath+'" data-setup="{}"></video>');
            var src = '<source src="'+videoUrl+'" type="video/mp4" />';
            var videoCaption = '<caption class="yhnCaption" data-type="'+this.props.videoTitle+'" data-title="'+imgTitle+'" data-alt="'+imgAlt+'">'+imgDesc+'</caption>';
            video.append(src);
            try {
                return videoWrap.append(videoButton, video, videoCaption);
            } finally {
                imgViewPath = null;
                imgThumbPath = null;
                imgTitle = null;
                imgDesc = null;
                imgAlt = null;
                videoSrcArray = null;
                videoButton = null;
                video = null;
                videoCaption = null;
            }
        },
        getSlideMainPhotoDom : function(node){
            var imgViewPath = node.attr('data-viewpath');
            var imgTitle = node.attr('data-title');
            var imgAlt = node.attr('data-alt');
            if( $.trim(imgTitle).length < 1 ) imgTitle = $('meta[name="title"]').attr('content');
            var imgDesc = node.attr('data-caption');
            var photoWrap = $('<div class="img-box">' +
                '<figure><img src="" alt="">' +
                '<figcaption class="yhnCaption" data-type="'+this.props.imageTitle+'" data-title="'+imgTitle+'">'+imgDesc+'</figcaption></figure>' +
                '<button type="button" class="btn-prev02">previous</button>' +
                '<button type="button" class="btn-next02">next</button>' +
                '</div>');
            photoWrap.find('img').attr({'src' : imgViewPath, 'alt' : imgAlt});
            try {
                return photoWrap;
            } finally {
                imgViewPath = null;
                imgTitle = null;
                imgDesc = null;
                imgAlt = null;
                photoWrap = null;
            }
        },
        getSlideMainContentDom : function(node){
            var contentType = node.attr('data-grouptype');
            var item = $('<div></div>');
            var itemBox;
            if( contentType === 'mpic' ){
                item.addClass('video-view-con yhnSlideItem');
                itemBox = this.getSlideMainVideoDom(node);
            } else {
                item.addClass('pto-view-con yhnSlideItem');
                itemBox = this.getSlideMainPhotoDom(node);
            }
            try {
                return item.append(itemBox)[0];
            } finally {
                contentType = null;
                item = null;
                itemBox = null;
            }
        },
        getSlideThumbContentDom : function(node){
            var contentType = node.attr('data-grouptype');
            var imgThumbPath = node.attr('data-thumnailpath');
            var thumbBox = $('<div class="thumb-slide"></div>');
            var figureBox = $('<figure class="img-con img-cover"></figure>');
            var imgBox = '<a href="#none"><img src="'+imgThumbPath+'" alt=""></a>';
            if( contentType === 'mpic' ){ figureBox.addClass('img-con1'); }
            try {
                thumbBox.append(figureBox.append(imgBox));
                return thumbBox[0];
            } finally {
                contentType = null;
                imgThumbPath = null;
                thumbBox = null;
                figureBox = null;
                imgBox = null;
            }
        },
        parseGroupObject : function(){
            $.each(this.group, $.proxy(function(key, val){
                var target = this.group[key][0];
                var insertParent = target.parentNode;
                var newDom = this.createSliderDom(key, val);
                if( this.props.isFadeIn ){newDom[0].style.opacity = 0;}
                newDom.attr('data-render', 'mCompiler');
                insertParent.insertBefore(newDom[0], target);
                if( val.length === 1 ){newDom.find('.thumb-box').hide();}
                if( this.props.isFadeIn ){newDom.animate({opacity : 1}, 2000 );}
                insertParent.removeChild(target);
            }, this));
        },
        removeGroupObject : function(){
            var module = this;
            $.each(this.group, function(key){
                var target = module.group[key];
                var i = 0;
                for(; i < target.length; i++){
                    var parent = target[i].parentNode;
                    if( parent && parent.parentNode ){
                        parent.parentNode.removeChild(parent);
                    }
                }
            });
        },
        onStartGroupItem : function(){
            var module = this;
            var selector = $(this.props.customTagName+this.props.groupClass);
            if( selector.length > 0 ){
                $.each(selector, function(num, val){
                    var groupName = $(val).attr('groupid');
                    if( groupName && groupName !== '' ){
                        var group = 'group'+groupName;
                        if( !module.group[group] ){ module.group[group] = [];}
                        module.group[group].push(this);
                    }
                });
            }
            this.parseGroupObject();
            return $('body').find('div[data-render="mCompiler"]');
        },
        init : function(props){
            $.extend(this.props, props);
            if( this.props.isModuleActive ){
                var result = this.onStartGroupItem();
                this.onVideoJsStart();
                this.onStartCaption(result);
                this.onStartLayout(result);
                if( LANG === 'AR' ){$('.ar .pic-zone .slide-article').css('direction' , 'ltr');}
            }
        }
    };
    MultiGroupCompiler.prototype.constructor = MultiGroupCompiler;
    window.YNA_SERVICE['MODULE']['MultiGroupCompiler'] = function(props){
        return new MultiGroupCompiler(props);
    };

    /** [PhotoGroupCompiler] Module Start */
    PhotoGroupCompiler = function(props){
        this.group = {};
        this.props = {
            isModuleActive: true,
            deviceType    : 'web',
            moduleType    : 'yna-img-slide',
            customTagName : 'yna',
            groupClass    : '.yna-img-slide',
            imageCodeName : 'image',
            imageTitle    : 'Photos',
            isFadeIn      : false,
            prevBtnClass  : '.btn-prev02',
            nextBtnClass  : '.btn-next02'
        };
        // 일본어 show : キャプションを読む, hide : キャプションを隠す
        this.langDef = {
            'show' : ['', 'show caption', '展开', '', 'مشاهدة التعليق', 'Ver el texto', 'Voir la légende'],
            'hide' :['', 'hide caption', '收起', '', 'إخفاء التعليق', 'Ocultar el texto', 'Cacher la legende']
        };
        this.init(props);
        return this.group;
    };
    PhotoGroupCompiler.prototype = {
        onStartLayout : function(mBox){
            var module = this;
            $.each(mBox, function(){
                var target = $(this);
                var items = target.find('.slide-article .yhnSlideItem');
                var firstItem = items.first();
                var lastItem = items.last();
                if( firstItem && lastItem ){
                    firstItem.find(module.props.prevBtnClass).remove();
                    lastItem.find(module.props.nextBtnClass).remove();
                }
            });
        },
        onStartCaption : function(mBox){
            $.each(mBox, function(){
                var target = $(this);
                var slideArticle = target.find('.slide-article .yhnSlideItem')[0];
                var capZone = target.find('.caption-zone');
                var targetCaptionData = $(slideArticle).find('.yhnCaption');
                capZone.find('p.desc').text(targetCaptionData.text());
                // .find('span.title').text(targetCaptionData.attr('data-title')).end()
                // .find('span.type').text(targetCaptionData.attr('data-type')).end()
            });
        },
        onVideoJsStart : function(){
            var video = $('body').find('video[data-name="'+this.props.videoDataName+'"]');
            if( window.videojs ){
                $.each(video, function(){ videojs(this.id); });
            } else {
                //console.log('[error] file Error - Check Please videoJs file!!');
            }
        },
        createSliderDom : function(key, array){
            var module = this;
            var dom = this.getSlideGroupDom(array);
            var mainSlide = [];
            var thumbSlide = [];
            dom.attr('data-group', key);
            $.each(array, function(num, node){
                var item = $(node);
                mainSlide.push(module.getSlideMainContentDom(item));
                thumbSlide.push(module.getSlideThumbContentDom(item));
            });
            dom.find('.slide-article').append($(mainSlide));
            dom.find('.thumb-imgs').append($(thumbSlide));
            try {
                return dom;
            } finally {
                mainSlide = null;
                thumbSlide = null;
            }
        },
        getSlideGroupDom : function(array){
            var totalLength = array.length;
            var dom = $('<div class="comp-box slider-group pic-zone" data-total="'+totalLength+'"></div>');
            var inner = $('<div class="inner"></div>');
            var slideArticle = '<div class="slide-article"></div>';
            var capZone = '<div class="caption-zone"><p class="desc"></p></div>';
            var button = '<button type="button" class="btn-caption btn"><span class="txt-show">'+this.langDef.show[LANG_NUM]+'</span><span class="txt-hide">'+this.langDef.hide[LANG_NUM]+'</span></button>';
            /** 썸네일 슬라이드 영역 */
            var thumbBox = $('<div class="thumb-box"></div>');
            var thumbControl = '<div class="control-box"><div class="controller"><button type="button" class="btn-prev">previous</button><button type="button" class="btn-next">next</button></div><span class="num">' +
                '<strong>1</strong><span>&nbsp;of&nbsp;</span><span> '+totalLength+'</span></span></div>';
            if( LANG_TYPE != 'ar' )
                var thumbSlide = '<div class="thumb-list-wrap"><div class="thumb-imgs slider-nav"></div></div>';
            else
                var thumbSlide = '<div class="thumb-list-wrap" dir="rtl"><div class="thumb-imgs slider-nav"></div></div>';
            thumbBox.append(thumbControl, thumbSlide);
            inner.append(slideArticle, capZone, button, thumbBox);
            dom.append(inner);
            
            var captionLength = 0;
            $.each(array, function(idx, item){
                var $item = $(item);
                captionLength += $.trim($item.attr("data-caption")).length;
            });
            if(!captionLength){
            	dom.find(".caption-zone p, .btn-caption").addClass("display-none");
            }
            if(totalLength<=1){
            	dom.find(".thumb-box").hide();
            }
            
            try {
                return dom;
            } finally {
                dom = null;
                inner = null;
                slideArticle = null;
                capZone = null;
                button = null;
                thumbBox = null;
                thumbControl = null;
                thumbSlide = null;
            }
        },
        getSlideMainPhotoDom : function(node){
            var imgViewPath = node.attr('data-viewpath');
            var imgTitle = node.attr('data-title');
            imgTitle = $.trim(imgTitle);
            var imgAlt = node.attr('data-alt');
            var imgDesc = node.attr('data-caption');
            var photoWrap = $('<div class="img-box">' +
                '<figure><img src="" alt="">' +
                '<figcaption class="yhnCaption" data-type="'+this.props.imageTitle+'" data-title="'+imgTitle+'" data-caption="'+imgDesc+'" data-alt="'+imgAlt+'">'+imgDesc+'</figcaption></figure>' +
                '<button type="button" class="btn-prev02">previous</button>' +
                '<button type="button" class="btn-next02">next</button>' +
                '</div>');
            photoWrap.find('img').attr({'src' : imgViewPath, 'alt' : imgAlt});
            try {
                return photoWrap;
            } finally {
                imgViewPath = null;
                imgTitle = null;
                imgDesc = null;
                imgAlt = null;
                photoWrap = null;
            }
        },
        getSlideMainContentDom : function(node){
            var item = $('<div></div>');
            var itemBox;
            item.addClass('pto-view-con yhnSlideItem');
            itemBox = this.getSlideMainPhotoDom(node);
            try {
                return item.append(itemBox)[0];
            } finally {
                item = null;
                itemBox = null;
            }
        },
        getSlideThumbContentDom : function(node){
            var imgThumbPath = node.attr('data-thumnailpath');
            var imgAlt = node.attr('data-alt');
            var thumbBox = $('<div class="thumb-slide"></div>');
            var figureBox = $('<figure class="img-con img-cover"></figure>');
            var imgBox = '<a href="#none"><img src="'+imgThumbPath+'" alt="'+imgAlt+'"></a>';
            try {
                thumbBox.append(figureBox.append(imgBox));
                return thumbBox[0];
            } finally {
                imgThumbPath = null;
                imgAlt = null;
                thumbBox = null;
                figureBox = null;
                imgBox = null;
            }
        },
        parseGroupObject : function(){
            $.each(this.group, $.proxy(function(key, val){
                var target = this.group[key][0];
                // <YNAOBJECT>가 단락 처리가 될 때가 있고 안될 때가 있음. 각각 처리해주기 위해 조건 추가
                if( $(target).parents('.comp-box').length < 1 ){
                    var insertParent = target.parentNode;
                    var newDom = this.createSliderDom(key, val);
                    if( this.props.isFadeIn ){newDom[0].style.opacity = 0;}
                    newDom.attr('data-render', 'pCompiler');
                    insertParent.insertBefore(newDom[0], target);
                    if( val.length === 1 ){newDom.find('.thumb-box').hide();}
                    if( this.props.isFadeIn ){newDom.animate({opacity : 1}, 2000 );}
                    insertParent.removeChild(target);
                } else {
                    var insertParent = $(target).parents('.comp-box');
                    var newDom = this.createSliderDom(key, val);
                    if( this.props.isFadeIn ){newDom[0].style.opacity = 0;}
                    newDom.attr('data-render', 'pCompiler');
                    insertParent.before(newDom[0]);
                    if( val.length === 1 ){newDom.find('.thumb-box').hide();}
                    if( this.props.isFadeIn ){newDom.animate({opacity : 1}, 2000 );}
                    insertParent.remove();
                }
            }, this));
        },
        removeGroupObject : function(){
            var module = this;
            $.each(this.group, function(key){
                var target = module.group[key];
                var i = 0;
                for(; i < target.length; i++){
                    var parent = target[i].parentNode;
                    if( parent && parent.parentNode ){
                        parent.parentNode.removeChild(parent);
                    }
                }
            });
        },
        onStartGroupItem : function(){
            var module = this;
            var selector = $(this.props.customTagName+this.props.groupClass);
            if( selector.length > 0 ){
                var group = 'photoGroup';
                if( !module.group[group] ){ module.group[group] = [];}
                $.each(selector, function(){
                    module.group[group].push(this);
                });
            }
            this.parseGroupObject();
            return $('body').find('div[data-render="pCompiler"]');
        },
        mobileTypeParse : function(result){
            /** 퓨즈에서 웹과 모바일을 다르게 만들었기때문에 급하게 모바일용을 따로 구현한다. */
            var langDef = this.langDef;
            var totalCount = 0;
            var articleCid = location.pathname.replace(/(.*)(\/view\/)([A-Z]{3}[0-9]{17})(.*)/,'$3');
            var slidePhotoViewUrl = '/related-photos/index?cid='+articleCid;
            if( location.hostname.match('cb.yna.co.kr') ) slidePhotoViewUrl = '/gate/big5/m-cn.yna.co.kr/related-photos/index?cid='+articleCid;
            var makeItem = function(param){
                var dom;
                if( param ){
                    var imgBox = '<div class="img-box"><figure><img src="'+param['imgSrc']+'" alt="'+param['alt']+'" data-caption="'+param['content']+'"></figure><a href="'+slidePhotoViewUrl+'" title="zoom in/out" class="btn-zoom">zoom in/out</a></div>';
                    dom = '<div class="inner swiper-slide">'+imgBox+'</div>';
                }
                return dom;
            };
            var capBox = '<div class="caption-zone"><p class="desc"></p></div>';
            var article = result.find('.slide-article .pto-view-con');
            var swiperTarget = $('<div class="comp-box slider-group photo-group"><div class="photo-module swiper-container"></div></div>');
            var swiperWrapper = $('<div class="inner-wrap swiper-wrapper"></div>');
            //var controlBtn = '<button type="button" class="btn-prev">previous</button><button type="button" class="btn-next">next</button>';
            var $paging = $('<div class="caption-control"></div>');
            if( article && article.length > 0 ){
                if(article.length>1){
                    $paging.append('<span class="paging"><strong>1</strong>&nbsp;of&nbsp;<span class="total">'+article.length+'</span></span>');
                }
                var titleLength = 0;
                var captionLength = 0;
                $.each(article, function(num){
                    var item = $(this);
                    var itemCap = item.find('figcaption');
                    var imgSrc = item.find('img').attr('src');
                    var title = itemCap.attr('data-title').replace(/\"/gi,"&quot;") || '';
                    var alt = itemCap.attr('data-alt').replace(/\"/gi,"&quot;");
                    var content = itemCap.text().replace(/\"/gi,"&quot;") || '';
                    var itemObj = {
                        imgSrc : imgSrc,
                        title : title,
                        alt : alt,
                        content : content,
                        nowLength : (num+1),
                        totalLength : article.length
                    };
                    titleLength += title.length;
                    captionLength += $.trim($(itemCap).text()).length;

                    var makeHtml = makeItem(itemObj);
                    if( makeHtml ){
                        swiperWrapper.append(makeHtml);
                    }
                });
                if(captionLength){
                    $paging.append('<button type="button" class="btn-caption"><span class="txt-show">'+langDef.show[LANG_NUM]+'</span><span class="txt-hide">'+langDef.hide[LANG_NUM]+'</span></button>');
                }
                // .append(controlBtn);
                swiperTarget.find('.swiper-container').html(swiperWrapper);
                swiperTarget.append(capBox+$paging[0].outerHTML);
                result.after(swiperTarget).remove();
            }
        },
        init : function(props){
            $.extend(this.props, props);
            if( this.props.isModuleActive ){
                var result = this.onStartGroupItem();
                // this.props.deviceType['currentDevice'] === 'mobile' , 모바일 기준X 도메인 기준으로 변경
                if( location.hostname.match('m-') || location.href.match('cb.yna.co.kr/gate/big5/m-cn.yna.co.kr') ){
                    this.mobileTypeParse(result);
                } else {
                    this.onStartCaption(result);
                    this.onStartLayout(result);
                    if( LANG === 'AR' ){$('.ar .pic-zone .slide-article').css('direction' , 'ltr');}
                }
            }
        }
    };
    PhotoGroupCompiler.prototype.constructor = PhotoGroupCompiler;
    window.YNA_SERVICE['MODULE']['PhotoGroupCompiler'] = function(props){
        return new PhotoGroupCompiler(props);
    };

    /** [VideoGroupCompiler] Module Start */
    VideoGroupCompiler = function(props){
        this.group = {};
        this.props = {
            isModuleActive: true,
            mediaUrl      : window.YNA_SERVICE['MODULE']['ynaCdnUrl'],
            customTagName : 'yna',
            groupClass    : '.yna-mpic-slide',
            videoCodeName : 'mpic',
            videoTitle    : 'Videos',
            isFadeIn      : false
        };

        this.langCaptionDef = {
            show : ['', 'show caption', '展开', 'キャプションを読む', 'مشاهدة التعليق', 'Ver el texto', 'Voir la légende'],
            hide :['', 'hide caption', '收起', 'キャプションを隠す', 'إخفاء التعليق', 'Ocultar el texto', 'Cacher la legende']
        };

        this.langVideoDef = {
            'show' : ['', 'show Related Video', '展开', '関連映像を見る', 'مشاهدة مقاطع الفيديو', 'Ver vídeos relacionados', 'Cacher les vidéos liées'],
            'hide' :['', 'Hide Related Video', '收起', '関連映像を隠す', 'اخفاء مقاطع الفيديو', 'Esconder vídeos relacionados', 'Montrer les vidéos liées']
        };
        this.init(props);
        return this.group;
    };
    VideoGroupCompiler.prototype = {
        onStartCaption : function(mBox){
            $.each(mBox, function(){
                var target = $(this);
                var capZone = target.find('.caption-zone');
                var targetCaptionData = target.find('.yhnCaption').first();
                if( targetCaptionData.attr('data-title').length > 0 ) capZone.find('span.title').text(targetCaptionData.attr('data-title'));
                else capZone.find('span.title').hide();
                if( targetCaptionData.text().length > 0 ) capZone.find('p.desc').text(targetCaptionData.text());
                else capZone.find('p.desc').hide();
                // find('span.type').text(targetCaptionData.attr('data-type')).end()
            });
        },
        onVideoJsStart : function(){
            //var video = $('body').find('video[data-name="'+this.props.videoDataName+'"]');
            var video = $('#todayVideoPlay');
            if( window.videojs ){
                $.each(video, function(){ videojs(this.id); });
            } else {
                /*
                 $('.btn-play').on('click',function () {
                 video.get(0).play();
                 return false;
                 });
                 */
                //console.log('[error] file Error - Check Please videoJs file!!');
            }
        },
        createSliderDom : function(key, array){
            var module = this;
            var dom = this.getSlideGroupDom(array);
            var mainSlide = [];
            var thumbSlide = [];
            dom.attr('data-group', key);
            mainSlide.push(module.getSlideMainContentDom($(array[0])));
            $.each(array, function(num, node){
                thumbSlide.push(module.getSlideThumbContentDom($(node)));
                if( num == 0 ) $(thumbSlide[num]).find('figure').addClass('on');
            });

            dom.find('.thumb-imgs').append($(thumbSlide));
            dom.find('.video-view-con').append($(mainSlide));

            try {
                return dom;
            } finally {
                mainSlide = null;
                thumbSlide = null;
            }
        },
        createMobileSliderDom : function(key, array){
            var module = this;
            var dom = this.getMobileSliderGroupDom(array);
            var thumbSlide = [];
            dom.outer.attr('data-group', key);
            var txt = '';
            var slideWidth = 0;
            module.getMobileSlideMainVideoDom($(array[0]), dom.mainVideoZone);
            $.each(array, function(num, node){
                thumbSlide.push(module.getMobileSlideThumbContentDom($(node), num, dom.swiperSlide));
                txt += thumbSlide[num][0].outerHTML;
            });

            dom.slideVideoZone.find('.swiper-wrapper').append(txt);
            dom.outer.append(dom.mainVideoZone, dom.slideVideoZone);
            return dom.outer;
        },
        getSlideGroupDom : function(array){
            var totalLength = array.length;
            var dom = $('<div class="comp-box slider-group video-group" data-total="'+totalLength+'"></div>');
            var videoZone = $('<div class="video-zone"></div>');
            var inner = $('<div class="inner"></div>');
            var videoBox = '<div class="video-view-con yhnSlideItem"></div>';
            var capZone = '<div class="caption-zone"><!--<strong><span class="type"></span>&nbsp;:&nbsp;</strong>--><span class="title" style="display:none"></span><p class="desc" style="display:none"></p></div>';
            var button = '<button type="button" class="btn-caption btn on"><span class="txt-show">'+this.langCaptionDef.show[LANG_NUM]+'</span><span class="txt-hide">'+this.langCaptionDef.hide[LANG_NUM]+'</span></button>';
            /** 썸네일 슬라이드 영역 */
            var thumbBox = $('<div class="thumb-box"></div>');
            var thumbControl = '<div class="control-box"><div class="controller"><button type="button" class="btn-prev">previous</button><button type="button" class="btn-next">next</button></div><span class="num">' +
                '<strong>1</strong><span>&nbsp;of&nbsp;</span><span> '+totalLength+'</span></span></div>';
            if( LANG_TYPE != 'ar' )
                var thumbSlide = '<div class="thumb-list-wrap"><div class="thumb-imgs"></div></div>';
            else
                var thumbSlide = '<div class="thumb-list-wrap" dir="rtl"><div class="thumb-imgs"></div></div>';
            thumbBox.append(thumbControl, thumbSlide);
            inner.append(videoBox, capZone, button, thumbBox);
            videoZone.append(inner);
            dom.append(videoZone);
            
            if(totalLength<=1){
            	dom.find(".btn-caption").hide();
            	dom.find(".thumb-box").hide();
            }
            
            try {
                return dom;
            } finally {
                dom = null;
                inner = null;
                videoBox = null;
                capZone = null;
                button = null;
                thumbBox = null;
                thumbControl = null;
                thumbSlide = null;
            }
        },
        getMobileSliderGroupDom : function(array){
            var totalLength = array.length;
            var outer = $('<div class="comp-box slider-group video-group"></div>');
            var mainVideoZone = $('<div class="video-module"></div>');
            var mainVideoImg = $('<div class="img-box"><a href=""><figure><img src="" alt=""></figure><button type="button" class="btn-play btn-video" data-layer="layerVideo" tabindex="0"><span>Play</span></button><span class="runtime"></span><a></div>');
            //var mainVideoCaption = $('<div class="caption-zone"><!--<strong>Video : </strong><span></span>--></div>');
            mainVideoZone.append(mainVideoImg);
            // swiper-container02
            var swiperContainer = $('<div class="thumb-list-wrap swiper-container02" id="relateVideo"></div>');
            var swiperWrapper = $('<div class="thumb-imgs swiper-wrapper"></div>');
            var swiperSlide = $('<div class="swiper-slide" title="" data-viewPath="" data-url=""><figure class="img-con img-cover"><img src="" alt=""></figure><div class="txt-con"></div></div>');
            var swiperScrollBar = $('<div class="swiper-scrollbar"></div>');
            var relatedHideBtn = $('<div class="related-control"><button type="button" class="btn-caption" data-open="relateVideo"><span class="txt-show">'+this.langVideoDef.show[LANG_NUM]+'</span><span class="txt-hide">'+this.langVideoDef.hide[LANG_NUM]+'</span></button><!-- Show Caption : class on 추가 --></div>');
            var slideVideoZone = $('<div class="thumb-box"></div>');
            swiperContainer.append(swiperWrapper, swiperScrollBar);
            slideVideoZone.append(relatedHideBtn, swiperContainer);

            if(array.length <= 1){
                slideVideoZone.addClass("display-none");
            }

            var dom = {
                outer : outer,
                mainVideoZone : mainVideoZone,
                slideVideoZone : slideVideoZone,
                swiperSlide : swiperSlide
            };
            return dom;
        },
        getSlideMainVideoDom : function(node){
            var imgViewPath = node.attr('data-viewpath').replace('_P2','_P4');
            var imgThumbPath = node.attr('data-thumnailpath').replace('_T','_P4');
            var videoSrcArray = imgThumbPath.split('.');
            var id = node.attr('id');
            var idDate = id.substr(3,6);
            //var videoUrl = this.props.mediaUrl+idDate+'/'+id+'_700M1.mp4';
            var videoUrl = node.attr('data-cdnurl');
            var now = $.now();
            //TODO : var videoId = 'yhnVdo_'+now;
            var videoId = 'todayVideoPlay';
            var videoWrap = $('<div class="video"></div>');
            var video = $('<video id="'+videoId+'" data-name="'+this.props.videoDataName+'" class="video-js vjs-default-skin vid-skin02" controls preload="auto" width="750" height="420" poster="'+imgViewPath+'" data-setup="{}"></video>');
            var src = '<source src="'+videoUrl+'" type="video/mp4" />';
            video.append(src);
            try {
                return videoWrap.append(video);
            } finally {
                imgViewPath = null;
                imgThumbPath = null;
                videoSrcArray = null;
                video = null;
            }
        },
        getMobileSlideMainVideoDom : function(node, dom){
            var title = node.attr('data-title');
            var imgAlt = node.attr('data-alt');
            var imgViewPath = node.attr("data-viewPath").replace("_P2","_P4");
            var imgThumbPath = node.attr('data-thumnailpath');
            var videoSrcArray = imgThumbPath.split('.');
            var duration = node.attr('data-duration');
            var id = node.attr('id');

            var url = '/view/'+id;
            if( location.hostname.match('cb.yna.co.kr') ) url = '//cb.yna.co.kr/gate/big5/m-cn.yna.co.kr/view/'+id;
            dom.find('.img-box a').attr("href", url);
            dom.find('.img-box img').attr('alt', imgAlt);
            dom.find('.img-box img').attr('src', imgViewPath);
            dom.find('.runtime').text(duration);
            dom.find('.caption-zone span').text(title);
        },
        getSlideMainContentDom : function(node){
            var itemBox;
            try {
                return this.getSlideMainVideoDom(node)[0];
            } finally {
                itemBox = null;
            }
        },
        getSlideThumbContentDom : function(node, num){
            var imgTitle = node.attr('data-title');
            imgTitle = $.trim(imgTitle);
            var imgAlt = node.attr('data-alt');
            var imgDesc = node.attr('data-caption');
            var id = node.attr('id');
            var idDate = id.substr(3,6);
            //var videoUrl = this.props.mediaUrl+idDate+'/'+id+'_700M1.mp4';
            var videoUrl = node.attr('data-cdnurl');
            var imgThumbPath = node.attr('data-thumnailpath').replace('_T','_P4');
            var thumbBox = $('<div class="thum-slide"></div>');
            var figureBox = $('<figure class="img-con"></figure>');
            var imgBox = '<a data-video="'+videoUrl+'"><img src="'+imgThumbPath+'" alt="'+imgAlt+'"></a>';
            var titleBox = '<div class="txt-con"><a href="#" tabindex="'+num+'">'+imgTitle+'</a></div>';
            var videoCaption = '<caption class="yhnCaption" data-type="'+this.props.videoTitle+'" data-title="'+imgTitle+'">'+imgDesc+'</caption>';
            try {
                thumbBox.append(figureBox.append(imgBox), titleBox, videoCaption);
                return thumbBox[0];
            } finally {
                imgDesc = null;
                imgThumbPath = null;
                thumbBox = null;
                figureBox = null;
                imgBox = null;
            }
        },
        getMobileSlideThumbContentDom : function(node, num, dom){
            var slide = dom;
            var imgTitle = node.attr('data-title');
            imgTitle = $.trim(imgTitle);
            var imgAlt = node.attr('data-alt');
            var imgDesc = node.attr('data-caption');
            var id = node.attr('id');
            var idDate = id.substr(3,6);
            var videoUrl = node.attr('data-cdnurl');
            var imgThumbPath = node.attr('data-thumnailpath');
            var url = '/view/'+id;
            var imgViewPath = node.attr("data-viewPath").replace("_P2","_P4");
            slide.attr("title",imgTitle);
            slide.attr("data-viewPath",imgViewPath);
            slide.attr("data-url",url);
            slide.find('.img-con img').attr('src', imgThumbPath);
            slide.find('.img-con img').attr('alt', imgAlt);
            slide.find('.txt-con').text(imgTitle);
            return slide;
        },
        parseGroupObject : function(){
            if( !location.hostname.match('m-') && !location.href.match('cb.yna.co.kr/gate/big5/m-cn.yna.co.kr') ){
                $.each(this.group, $.proxy(function(key, val){
                    var target = this.group[key][0];
                    var insertParent = target.parentNode;
                    if( $(target).parents('.comp-box').length ) var insertParent = $(target).parents('.comp-box');
                    var newDom = this.createSliderDom(key, val);
                    if( this.props.isFadeIn ){newDom[0].style.opacity = 0;}
                    newDom.attr('data-render', 'vCompiler');
                    if( $(target).parents('.comp-box').length ) insertParent.before(newDom[0]);
                    else insertParent.insertBefore(newDom[0], target);
                    if( val.length === 1 ){newDom.find('.thumb-box').hide();}
                    if( this.props.isFadeIn ){newDom.animate({opacity : 1}, 2000 );}
                    if( $(target).parents('.comp-box').length ) insertParent.remove();
                    else insertParent.removeChild(target);
                }, this));
            }else{
                $.each(this.group, $.proxy(function(key, val){
                    var target = this.group[key][0];
                    var insertParent = target.parentNode;
                    if( $(target).parents('.comp-box').length ) var insertParent = $(target).parents('.comp-box');
                    var newDom = this.createMobileSliderDom(key, val);
                    if( this.props.isFadeIn ){newDom[0].style.opacity = 0;}
                    newDom.attr('data-render', 'vCompiler');
                    if( $(target).parents('.comp-box').length ) insertParent.before(newDom[0]);
                    else insertParent.insertBefore(newDom[0], target);
                    if( val.length === 1 ){newDom.find('.thumb-box').hide();}
                    if( this.props.isFadeIn ){newDom.animate({opacity : 1}, 2000 );}
                    if( $(target).parents('.comp-box').length ) insertParent.remove();
                    else insertParent.removeChild(target);
                }, this));
            }
        },
        removeGroupObject : function(){
            var module = this;
            $.each(this.group, function(key){
                var target = module.group[key];
                var i = 0;
                for(; i < target.length; i++){
                    var parent = target[i].parentNode;
                    if( parent && parent.parentNode ){
                        parent.parentNode.removeChild(parent);
                    }
                }
            });
        },
        onStartGroupItem : function(){
            var module = this;
            var selector = $(this.props.customTagName+this.props.groupClass);
            if( selector.length > 0 ){
                var group = 'videoGroup';
                if( !module.group[group] ){ module.group[group] = [];}
                $.each(selector, function(){
                    module.group[group].push(this);
                });
            }
            this.parseGroupObject();
            return $('body').find('div[data-render="vCompiler"]');
        },
        init : function(props){
            $.extend(this.props, props);
            if( this.props.isModuleActive ){
                var result = this.onStartGroupItem();
                if( location.hostname.match('m-') || location.href.match('cb.yna.co.kr/gate/big5/m-cn.yna.co.kr') ){
                    if( LANG === 'AR' ){$('.ar .pic-zone .slide-article').css('direction' , 'ltr');}
                    imgCrop ? imgCrop() : '';
                }else{
                    this.onStartCaption(result);
                    var container = $('.thumb-list-wrap');
                    var wrapper = $('.thumb-list-wrap .swiper-wrapper');
                    var slideWidth = 0;
                    $.each(wrapper.find('.swiper-slide'), function(idx, li){
                        slideWidth += $(li).width();
                    });
                    var margin = $('.text-group').outerWidth(true) - $('.text-group').outerWidth();
                    wrapper.css('width', slideWidth+margin+'px');
                }
            }
        }
    };
    VideoGroupCompiler.prototype.constructor = VideoGroupCompiler;
    window.YNA_SERVICE['MODULE']['VideoGroupCompiler'] = function(props){
        return new VideoGroupCompiler(props);
    };

    /** [Photo IPT Compiler] Module Start */
    PhotoIPTCompiler = function(props){
        this.items = null;
        this.props = {
            isModuleActive: true,
            customTagName : 'yna',
            thumbString   : 'T',
            mainImgString : 'P2',
            groupClass    : '.yna-ipt-slide',
            imageCodeName : 'image',
            imageTitle    : 'Photos',
            prevBtnClass  : '.btn-prev02',
            nextBtnClass  : '.btn-next02',
            urlObject     : null,
            isFadeIn      : false,
            domainParse   : null,
            request       : null
        };
        this.langDef = {
            show : ['', 'show caption', '展开', 'キャプションを読む', 'مشاهدة التعليق', 'Ver el texto', 'Voir la légende'],
            hide :['', 'hide caption', '收起', 'キャプションを隠す', 'إخفاء التعليق', 'Ocultar el texto', 'Cacher la legende']
        };
        this.init(props);
        return this.items;
    };
    PhotoIPTCompiler.prototype = {
        onStartLayout : function(mBox){
            var module = this;
            $.each(mBox, function(){
                var target = $(this);
                var items = target.find('.slide-article .yhnSlideItem');
                var firstItem = items.first();
                var lastItem = items.last();
                if( firstItem && lastItem ){
                    firstItem.find(module.props.prevBtnClass).remove();
                    lastItem.find(module.props.nextBtnClass).remove();
                }
            });
        },
        onStartCaption : function(mBox){
            $.each(mBox, function(){
                var target = $(this);
                var slideArticle = target.find('.slide-article .yhnSlideItem')[0];
                var capZone = target.find('.caption-zone');
                var targetCaptionData = $(slideArticle).find('.yhnCaption');
                capZone.find('span.title').text(targetCaptionData.attr('data-title')).end()
                    .find('p.desc').text(targetCaptionData.text());
                //.find('span.type').text(targetCaptionData.attr('data-type')).end()
            });
        },
        removeGroupObject : function(){
            $.each( this.items, function(){
                var parent = this.parentNode;
                if( parent ){parent.removeChild(this);}
            });
        },
        getSlideThumbContentDom : function(obj){
            var thumbBox = $('<div class="thumb-slide"></div>');
            var figureBox = $('<figure class="img-con img-cover"></figure>');
            var imgDomainUrl = this.props.domainParse(obj['IMG']);
            var imgThumbPath = '//'+imgDomainUrl+obj['IMG'];
            var imgAlt = obj['TITLE'];
            var imgBox = '<a href="#none"><img src="'+imgThumbPath+'" alt="'+imgAlt+'"></a>';
            try {
                thumbBox.append(figureBox.append(imgBox));
                return thumbBox[0];
            } finally {
                imgThumbPath = null;
                thumbBox = null;
                figureBox = null;
                imgBox = null;
            }
        },
        getSlideMainPhotoDom : function(obj){
            var imgDomainUrl = this.props.domainParse(obj['IMG']);
            var imgViewPath = imgDomainUrl+obj['IMG'];
            var imgTitle = obj['TITLE'] || $('meta[name="title"]').attr('content');
            var imgDesc = obj['LINK_CAPTION'] || '';
            var imgAlt = obj['TITLE'] || obj['LINK_CAPTION'];
            var imgType = imgViewPath.split('.');
            var thumbString = '_'+this.props.thumbString;
            var mainImgString = '_'+this.props.mainImgString;
            imgType = imgType[imgType.length-1];
            imgViewPath = '//'+(imgViewPath.split(thumbString))[0]+mainImgString+'.'+imgType;
            var photoWrap = $('<div class="img-box">' +
                '<figure><img src="" alt="">' +
                '<figcaption class="yhnCaption" ' +
                'data-type="'+this.props.imageTitle+'" ' +
                'data-title="'+imgTitle+'" data-alt="'+imgAlt+'">'+imgDesc+'</figcaption></figure>' +
                '<button type="button" class="btn-prev02">previous</button>' +
                '<button type="button" class="btn-next02">next</button>' +
                '</div>');
            photoWrap.find('img').attr({'src' : imgViewPath, 'alt' : imgAlt});
            try {
                return photoWrap;
            } finally {
                imgViewPath = null;
                imgTitle = null;
                imgDesc = null;
                imgAlt = null;
                photoWrap = null;
            }
        },
        getSlideMainContentDom : function(obj){
            var item = $('<div></div>');
            var itemBox;
            item.addClass('pto-view-con yhnSlideItem');
            itemBox = this.getSlideMainPhotoDom(obj);
            try {
                return item.append(itemBox)[0];
            } finally {
                item = null;
                itemBox = null;
            }
        },
        getSlideMobileContentDom : function(obj){
            var item = $('<div class="inner swiper-slide"><div class="img-box"><figure></figure></div></div>');
            var imgDomainUrl = this.props.domainParse(obj['IMG']);
            var imgViewPath = imgDomainUrl+obj['IMG'].replace('_T', '_P2');
            var imgDesc = obj['LINK_CAPTION'] || '';
            var imgAlt = obj['TITLE'] || obj['LINK_CAPTION'];
            var iptLink = (this.props.urlObject['origin'].indexOf('cb')<0?this.props.urlObject['origin']:this.props.urlObject['origin']+'/m-cn.yna.co.kr')+'/view/'+obj['CID'];
            item.find('figure').append($('<img src="'+imgViewPath+'" alt="'+imgAlt+'" data-caption="'+imgDesc+'">'));
            item.find('figure').after($('<a href="'+iptLink+'" title="zoom in/out" class="btn-zoom">zoom in/out</a>'));
            try{
                return item;
            } finally {
                item = null;
            }
        },
        getSlideGroupDom : function(array){
            var totalLength = array.length;
            var dom = $('<div class="comp-box slider-group pic-zone" data-total="'+totalLength+'"></div>');
            var inner = $('<div class="inner"></div>');
            var slideArticle = '<div class="slide-article"></div>';
            var capZone = '<div class="caption-zone"><p class="desc"></p></div>';
            var button = '<button type="button" class="btn-caption btn"><span class="txt-show">'+this.langDef.show[LANG_NUM]+'</span><span class="txt-hide">'+this.langDef.hide[LANG_NUM]+'</span></button>';
            /** 썸네일 슬라이드 영역 */
            var thumbBox = $('<div class="thumb-box"></div>');
            var thumbControl = '<div class="control-box"><div class="controller"><button type="button" class="btn-prev">previous</button><button type="button" class="btn-next">next</button></div><span class="num">' +
                '<strong>1</strong><span>&nbsp;of&nbsp;</span><span> '+totalLength+'</span></span></div>';
            if( LANG_TYPE != 'ar' ){
                var thumbSlide = '<div class="thumb-list-wrap"><div class="thumb-imgs slider-nav"></div></div>';
            }
            else {
                var thumbSlide = '<div class="thumb-list-wrap" dir="rtl"><div class="thumb-imgs slider-nav"></div></div>';
            }
            thumbBox.append(thumbControl, thumbSlide);
            inner.append(slideArticle, capZone, button, thumbBox);
            dom.append(inner);

            try {
                return dom;
            } finally {
                dom = null;
                inner = null;
                slideArticle = null;
                capZone = null;
                button = null;
                thumbBox = null;
                thumbControl = null;
                thumbSlide = null;
            }
        },
        getSliderGroupMobileDom : function(array){
            var totalLength = array.length;
            var dom = $('<div class="comp-box slider-group photo-group" data-total="'+totalLength+'"></div>');
            var container = $('<div class="photo-module swiper-container"></div>');
            var capZone = $('<div class="caption-zone"><p class="desc"></p></div>');
            var capControl = $('<div class="caption-control"></div>');
            var capPaging = $('<span class="paging"><strong></strong>&nbsp;of&nbsp;<span class="total"></span></span>');
            var capButton = $('<button type="button" class="btn-caption"><span class="txt-show">'+this.langDef.show[LANG_NUM]+'</span><span class="txt-hide">'+this.langDef.hide[LANG_NUM]+'</span></button>');
            var inner = $('<div class="inner-wrap swiper-wrapper"></div>');
            capControl.append(capPaging, capButton);
            container.append(inner);
            dom.append(container);
            dom.append(capZone, capControl);

            try {
                return dom;
            } finally {
                dom = null;
                container = null;
                inner = null;
            }
        },
        createSliderDom : function(key, array){
            var module = this;
            var dom;
            var mainSlide = [];
            var thumbSlide = [];
            if( array.length > 0 ){
                if( module.props.urlObject['href'].indexOf('m-') < 0 ) {
                    /* Web */
                    dom = this.getSlideGroupDom(array);
                    dom.attr('data-group', key);
                    $.each(array, function(num, obj){
                        mainSlide.push(module.getSlideMainContentDom(obj));
                        thumbSlide.push(module.getSlideThumbContentDom(obj));
                    });
                    dom.find('.slide-article').append($(mainSlide));
                    dom.find('.thumb-imgs').append($(thumbSlide));
                }
                else {
                    /* Mobile */
                    dom = this.getSliderGroupMobileDom(array);
                    dom.attr('data-group', key);
                    $.each(array, function(num, obj){
                        mainSlide.push(module.getSlideMobileContentDom(obj));
                    });
                    dom.find('.swiper-wrapper').append(mainSlide);
                    var total = dom.attr('data-total');
                    dom.find('.caption-control .paging strong').text(1);
                    dom.find('.caption-control .paging .total').text(total);
                }
            }
            try {
                return dom;
            } finally {
                mainSlide = null;
                thumbSlide = null;
            }
        },
        setIptContent : function(target, param){
            if(param && param.length > 0){
                var key = 'iptGroup';
                var insertParent = target.parentNode; //현재 아이템의 부모 p
                var newDom = this.createSliderDom(key, param); //전체돔
                if( this.props.isFadeIn ){newDom[0].style.opacity = 0;}
                newDom.attr('data-render', 'iptCompiler');
                insertParent.insertBefore(newDom[0], target);
                if( param.length === 1 ){newDom.find('.thumb-box').hide();}
                if( this.props.isFadeIn ){newDom.animate({opacity : 1}, 2000 );}
            }
        },
        onStartIptItem : function(){
            var module = this;
            var selector = this.items = $(this.props.customTagName+this.props.groupClass);
            if( selector.length > 0 ){
                $.each(selector, function(idx, s){
                    var attr = this.attributes;
                    var newObj = {};
                    $.each(attr, function(num, val){
                        newObj[val['name']] = val['value'];
                    });
                    /* 첫번째 화보 소재만 렌더링하고 그외 소재는 제거한다. */
                    if( idx == 0 ) { module.setIptRequest(this, newObj); }
                    else { $(this).remove(); }
                });
            }
            return $('body').find('div[data-render="iptCompiler"]');
        },
        setIptRequest : function(item, data){
            if( typeof this.props.request === 'function' ){
                typeof this.props.request(this, item, data);
            }
        },
        init : function(props){
            $.extend(this.props, props);
            if(this.props.isModuleActive ){
                var result = this.onStartIptItem();
                if( result.length > 0 ){
                    this.onStartCaption(result);
                    this.onStartLayout(result);
                }
                if( LANG == 'AR' && props.urlObject['host'].indexOf('m-') >= 0 ){ $('.ar .pic-zone .slide-article').css('direction' , 'ltr'); }
                this.removeGroupObject();
            }
        }
    };
    PhotoIPTCompiler.prototype.constructor = PhotoIPTCompiler;
    window.YNA_SERVICE['MODULE']['PhotoIPTCompiler'] = function(props){
        return new PhotoIPTCompiler(props);
    };

    /** [SocialGroupCompiler] Module Start */
    SocialGroupCompiler = function(props){
        this.group = {};
        this.props = {
            isModuleActive : true,
            customTagName : 'yna',
            groupClass    : '.yna-sns',
            totalFacebook : props.totalFacebook,
            totalTwitter : props.totalTwitter,
            totalInstagram : props.totalInstagram,
            facebookCnt : 0,
            twitterCnt : 0,
            instaCnt : 0
        };
        this.init(props);
        return this.group;
    };
    SocialGroupCompiler.prototype = {
        onStartGroupItem : function(){
            var module = this;
            var selector = $(this.props.customTagName+this.props.groupClass);
            if( selector.length > 0 ){
                var group = 'socialGroup';
                if( !module.group[group] ){ module.group[group] = [];}
                $.each(selector, function(){
                    module.group[group].push(this);
                });
            }
            this.parseGroupObject();
            return $('body').find('div[data-render="socialCompiler"]');
        },
        parseGroupObject : function(){
            $.each(this.group['socialGroup'], $.proxy(function(key, val){
                var target = this.group['socialGroup'][key];
                var insertParent = target.parentNode;
                var newDom = this.createSocialDom(key, val);
                if( newDom == null ) return true;
                newDom.attr('data-render', 'socialCompiler');
                insertParent.insertBefore(newDom[0], target);
                insertParent.removeChild(target);
            }, this));
        },
        createSocialDom : function(key, val){
            var module = this;
            var source = $(val).attr('data-source');
            if ( source == 'facebook' ) {
                var dom = module.createFacebookPost($(val));
            } else if ( source == 'twitter' ) {
                var dom = module.createTwitterPost($(val));
            } else if ( source == 'instagram' ) {
                var dom = module.createInstagramDom($(val));
            }
            try {
                dom.addClass('social-zone');
                return dom;
            } catch(e) {
                return dom = null;
            }
        },
        insertScript : function(source){
            var module = this;
            var id = source;
            var chk = true;
            if ( source == 'facebook-jssdk' ) {
                var src = 'https://connect.facebook.net/en_US/sdk.js';
            } else if ( source == 'twitter-widget-js' ) {
                var src = 'https://platform.twitter.com/widgets.js';
            } else if ( source == 'instagram-embeded-js' ) {
                var src = 'https://platform.instagram.com/en_US/embeds.js';
            }
            var js, fjs = document.getElementsByTagName('script')[0];
            $.each(document.getElementsByTagName('script'), function(){
                if( $(this).attr('id') == id ) {
                    chk = false;
                    module.callback(source);
                    return false;
                }
            });
            if( chk == false ) return;
            js = document.createElement('script');
            js.id = id;
            js.src = src;
            js.async = 'true';
            js.onload = function(){
                module.callback(source);
            };
            fjs.parentNode.insertBefore(js, fjs);
        },
        callback : function(source) {
            if( source == 'facebook-jssdk' ) {
                FB.init({
                	xfbml      : true,
                    version    : 'v3.2'
                });
            }
            else if( source == 'twitter-widget-js' ) {
                var target = $('.tw-embed div');
                $.each(target, function(idx){
                    var id = $(this).attr('data-id');
                    window.twttr.widgets.createTweet(
                        id,
                        document.getElementById('tw-embed'+idx),
                        { theme: 'white'}
                    );
                });
            }
            else if( source == 'instagram-embeded-js' ) {
                $('#instagram-embeded-js').onload = function(){
                    instgrm.Embeds.process();
                };
            }
        },
        createFacebookPost : function(val){
            var url = $(val).attr('data-url');
            var width = $(val).attr('data-width');
            var height = $(val).attr('data-height');
            var dom = $('<div class="fb-embed"></div>');
            var inner = $('<div class="fb-post" data-href="'+url+'"></div>');
            if( width != undefined ) { inner.attr('data-width', width); }
            if( height != undefined ) { inner.attr('data-height', height); }
            dom.append(inner);
            this.props.facebookCnt++;
            try {
                return dom;
            } finally {
                dom = null;
            }
        },
        createTwitterPost : function(val){
            var id = $(val).attr('data-id');
            if( !/MSIE ([6789]|10)/.test(navigator.userAgent) ) {
                var dom = $('<div class="tw-embed"></div>');
                var inner = $('<div id="tw-embed'+this.props.twitterCnt+'" data-id="'+id+'"></div>');
                dom.append(inner);
                this.props.twitterCnt++;
            }
            else {
                var url = $(val).attr('data-url');
                var dom = $('<div class="tw-link-post"><a href="'+url+'">'+url+'</a></div>');
            }
            try {
                return dom;
            } finally {
                dom = null;
            }
        },
        createInstagramDom : function(val){
            var id = $(val).attr('data-id');
            var width = $(val).attr('data-width');
            var dom = $('<div class="insta-embed"></div>');
            var inner = $('<blockquote data-id="'+id+'" class="instagram-media" data-instgrm-captioned="" data-instgrm-permalink="https://www.instagram.com/p/'+id+'/?utm_source=ig_embed&utm_campaign=embed_loading_state_script" data-instgrm-version="10" style=" background:#FFF; border:0; border-radius:3px; box-shadow:0 0 1px 0 rgba(0,0,0,0.5),0 1px 10px 0 rgba(0,0,0,0.15); max-width:552px; margin: 1px; padding:0; width:99.375%; width:-webkit-calc(100% - 2px);"></blockquote>');
            if( width != undefined ) { inner.find('.instagram-media').css('max-width', width); }
            dom.append(inner);
            this.props.instaCnt++;
            try {
                return dom;
            } finally {
                dom = null;
            }
        },
        init : function(props){
            $.extend(this.props, props);
            if( this.props.isModuleActive ){
                var result = this.onStartGroupItem();
                if ( this.props.totalFacebook != 0 && this.props.totalFacebook == this.props.facebookCnt ) {
                    this.insertScript('facebook-jssdk');
                }
                if ( this.props.totalTwitter != 0 && this.props.totalTwitter == this.props.twitterCnt ) {
                    this.insertScript('twitter-widget-js');
                }
                if ( this.props.totalInstagram != 0 && this.props.totalInstagram == this.props.instaCnt ) {
                    this.insertScript('instagram-embeded-js');
                }
            }
        }
    };
    SocialGroupCompiler.prototype.constructor = SocialGroupCompiler;
    window.YNA_SERVICE['MODULE']['SocialGroupCompiler'] = function(props){
        return new SocialGroupCompiler(props);
    };

    /** [HyperGroupCompiler] Module Start */
    HyperGroupCompiler = function(props){
        this.group = {};
        this.props = {
            isModuleActive : true,
            customTagName : 'yna',
            groupClass    : '.yna-hyper'
        };
        this.init(props);
        return this.group;
    };
    HyperGroupCompiler.prototype = {
        onStartGroupItem : function(){
            var module = this;
            var selector = $(this.props.customTagName+this.props.groupClass);
            if( selector.length > 0 ){
                var group = 'hyperGroup';
                if( !module.group[group] ){ module.group[group] = [];}
                $.each(selector, function(){
                    module.group[group].push(this);
                });
            }
            this.parseGroupObject();
        },
        parseGroupObject : function(){
            $.each(this.group['hyperGroup'], $.proxy(function(key, val){
                var target = this.group['hyperGroup'][key];
                var insertParent = target.parentNode;
                var newDom = this.createHyperDom(key, val);
                if( newDom == null ) return true;
                insertParent.insertBefore(newDom[0], target);
                insertParent.removeChild(target);
            }, this));
        },
        createHyperDom : function(key, val){
            var module = this;
            var dom = null;
            var dattr = $(val).attr('data-dattr');
            var url = $(val).attr('data-url');
            var desc = $(val).attr('data-desc');
            var stat_code = $(val).attr('data-stat-code');
            var stat_section = $(val).attr('data-stat-section');
            /** dattr이 0 이면 단순 링크, 1이면 링크박스로 표시한다. */
            if( dattr == "0" ) {
                try{
                    dom = $('<a href="'+url+'" class="anc-link" target="_blank" data-render="hyperCompiler">'+(desc.length > 0 ? desc : url)+'</a>');
                } catch(e) {
                    dom = $('<a href="'+url+'" class="anc-link" target="_blank" data-render="hyperCompiler">'+url+'</a>');
                }
            } else {
                dom = $('<span class="comp-box snippet-group"><span class="inner"><span class="snippet-zone"><span class="txt-con"><span class="info-con"></span></span></span></span></span>');
                dom.find('.info-con').append('<p>'+desc+'</p><a href="'+url+'" class="link" data-render="hyperCompiler">'+url+'</a>');
            }
            /** stat-code, stat-section이 있는 경우 a 링크의 attr로 추가한다. */
            if( stat_code != undefined && stat_section != undefined ) {
                dom.find('a').attr('data-stat-code', stat_code);
                dom.find('a').attr('data-stat-section', '');
            }
            try {
                return dom;
            } catch(e) {
                return dom = null;
            }
        },
        setExternalLink : function(){
        	var hyperItem = $('[data-render="hyperCompiler"]');
        	$.each(hyperItem, function(idx, item){
        		var link = $(this).attr('href');
        		if( !link.match('.yna.co.kr') ){
        			$(this).attr('rel','nofollow');
        		}
        	});
        },
        init : function(props){
            $.extend(this.props, props);
            if( this.props.isModuleActive ){
                this.onStartGroupItem();
            }
            if( this.props.externalLink ){
            	this.setExternalLink();
            }
        }
    };
    HyperGroupCompiler.prototype.constructor = HyperGroupCompiler;
    window.YNA_SERVICE['MODULE']['HyperGroupCompiler'] = function(props){
        return new HyperGroupCompiler(props);
    };

    /** [광고 컴파일러] */
   // AdvertisementCompiler = function(props){
   //     this.props = props;
   //     this.state = { method : ['POST','GET'], advDom : null, advContent : [] };
   //     this.control = { body : 'body' };
   //     this.init();
   // };
   // AdvertisementCompiler.prototype = {
   //     onActiveDom : function(){
   //         this.control.body = $(this.control.body);
   //     },
   //     onAjaxSuccessCallData2 : function(){
   //         var findAdDom = $("."+this.props.paragraphClass);
   //         if( findAdDom.length === 0 ){return false;}
   //         if( this.props.ajaxTargetUrl){
   //             var module = this;
   //             var method = (this.state.method.indexOf( this.props.ajaxMethod ) !== -1)?this.props.ajaxMethod:'GET';
   //             var dataType = (this.props.ajaxDataType)?this.props.ajaxDataType:'html';
   //             var getUrl = this.props.ajaxTargetUrl;
   //             var userAjax;
   //             getUrl = encodeURI(getUrl);
   //             userAjax = $.ajax({
   //                 url : getUrl,
   //                 method : 'GET',
   //                 //dataType : dataType,
   //                 cache : true,
   //                 success : function(data){}
   //             });
   //             userAjax.done(function(data) {
   //                 module.onCreateAdDom(data['DATA']);
   //                 /** 컨텐츠 높이 계산 */
   //                 window.YNA_SERVICE['RESIZE']();
   //             }).fail(function(err){
   //                 module.userError(err);
   //             }).always(function(){
   //                 //module.removeCustomDom();
   //             });
   //         }
   //     },
   //     onCreateAdDom : function(data){
   //         if( data ){
   //             var module = this;
   //             var checkParagraph = [];
   //             var checkAllParagraph = module.control.body.find('p.yna-p');
   //             $.each(data, function(num, obj){
   //                 var position = parseInt(obj['PARAGRAPH']);
   //                 checkParagraph[num] = checkAllParagraph[position-1];
   //             });
   //             $.each(data, function(num, obj){
   //                 var className = module.paragraphClass || 'yna-p';
   //                 var adId = obj['AD_ID'];
   //                 var useYn = obj['USE_YN'];
   //                 var periodYn = obj['PERIOD_YN'];
   //                 var paragraph = parseInt(obj['PARAGRAPH']);
   //                 var body = obj['BODY'];
   //                 var newAdDom = $('<p class="'+className+'" data-adid="'+adId+'"></p>');
   //                 newAdDom.html(body);
   //                 if( useYn !== 'Y' ){
   //                     newAdDom.html('<!--광고 중지되었습니다-->');
   //                 }
   //                 if( periodYn !== 'Y' ){
   //                     newAdDom.html('<!--서비스 기간이 아닙니다.-->');
   //                 }
   //                 module.onAppendContentFromAjax(paragraph, newAdDom, checkParagraph[num]);
   //             });
   //         }
   //     },
   //     onAppendContentFromAjax : function(position, html, target){
   //         if( html && position){
   //             if( this.props.isFadeIn ){
   //                 html = $(html).css({'opacity' : 0});
   //             }
   //             if( target ){
   //                 $(target).after(html);
   //             }
   //             if( this.props.isFadeIn ){
   //                 html.animate({opacity : 1}, 2000 );
   //             }
   //         }
   //     },
   //     onCreateParamData : function(param){
   //         return {
   //             domainid : param.domainId,
   //             siteid : param.siteId,
   //             id : param.id,
   //             textype : this.props.texTypeName || 'yna_component'
   //         };
   //     },
   //     removeCustomDom : function(){
   //         var module = this;
   //         $.each(this.state.advDom, function(){
   //             var item = $(this);
   //             var isParent = item.parent();
   //             var paragraphClass = module.props.paragraphClass.split('.')[1] || 'yna-p';
   //             if( isParent.hasClass(paragraphClass) && isParent.children().length === 1 ){isParent.remove();}
   //         })
   //     },
   //     getQueryString : function(param){
   //         return this.props.ajaxTargetUrl+'?' +
   //             'domainid=' + param.domainid +
   //             '&siteid=' + param.siteid +
   //             '&id=' + param.id +
   //             '&textype=' + param.textype;
   //     },
   //     userError : function(err){
   //         //console.log('[error] Ajax Error - Check Please Ajax Url Or Data!!');
   //     },
   //     init : function(){
   //         var module = this;
   //         $(function(){
   //             module.onActiveDom();
   //             module.onAjaxSuccessCallData2();
   //         })
   //     },
   //     onAjaxSuccessCallData : function(param, position){
   //         if( this.props.ajaxTargetUrl && param ){
   //             var module = this;
   //             var method = (this.state.method.indexOf( this.props.ajaxMethod ) !== -1)?this.props.ajaxMethod:'GET';
   //             var dataType = (this.props.ajaxDataType)?this.props.ajaxDataType:'html';
   //             var getUrl = this.props.ajaxTargetUrl;
   //             var userAjax;
   //             if( method === 'GET'){
   //                 getUrl = this.getQueryString(param);
   //             }
   //             getUrl = encodeURI(getUrl);
   //             userAjax = $.ajax({
   //                 url : getUrl,
   //                 method : method,
   //                 dataType : dataType,
   //                 cache : true,
   //                 success : function(data){}
   //             });
   //             userAjax.done(function(html) {
   //                 module.state.advContent.push( { paragraphNum : position, content : html } );
   //                 module.onAppendContentFromAjax(position, html);
   //                 /** 컨텐츠 높이 계산 */
   //                 window.YNA_SERVICE['RESIZE']();
   //             }).fail(function(err){
   //                 module.userError(err);
   //             }).always(function(){
   //                 module.removeCustomDom();
   //             });
   //         }
   //     },
   //     onCheckUserAdvContent : function(){
   //
   //         var props = this.props;
   //         var userTag = props.customTagName;
   //         var className = props.advContentClass;
   //         this.state.advDom = $(userTag+className);
   //
   //     },
   //     onCompileContentParse : function(paraArray, idArray, domainId, siteId){
   //         if( paraArray ){
   //             var module = this;
   //             $.each(paraArray, function(num, pos){
   //                 var completeData = module.onCreateParamData({
   //                     domainId : domainId,
   //                     siteId : siteId,
   //                     id : idArray[num] || idArray[0]
   //                 });
   //                 if( completeData ){
   //                     //module.onAjaxSuccessCallData(completeData, pos);
   //                 }
   //             });
   //         }
   //     },
   //     onCompileContentStart : function(){
   //         var module = this;
   //         $.each(this.state.advDom, function(){
   //             var item = $(this);
   //
   //             var advId = item.attr('adid');
   //             var domainId = item.attr('domainid');
   //             var targetParagraph = item.attr('paragrap');
   //             var siteId = item.attr('siteid');
   //
   //             /** string check */
   //             advId = ( advId )?advId.replace(/\s/gi, ""):advId;
   //             targetParagraph = ( targetParagraph )?targetParagraph.replace(/\s/gi, ""):targetParagraph;
   //
   //             /** splite array */
   //             advId = advId.split(',');
   //             targetParagraph = targetParagraph.split(',');
   //
   //             /** parse content */
   //             //module.onCompileContentParse(targetParagraph, advId, domainId, siteId);
   //         });
   //     }
   // };
   // AdvertisementCompiler.prototype.constructor = AdvertisementCompiler;
   // window.YNA_SERVICE['MODULE']['ADCompiler'] = function(props){ return new AdvertisementCompiler(props); };
   //
   //
   // /** [광고형/기간형] 컴포넌트 컴파일러 */
   // ComponentContentCompiler = function(props){
   //     this.props = props;
   //     this.state = {
   //         target : null
   //     };
   //     this.init();
   // };
   // ComponentContentCompiler.prototype = {
   //     findComponent : function(){
   //         var opt = this.props;
   //         var findDom = $('body').find(opt['customTagName']+opt['className']);
   //         if( findDom.length > 0 ){
   //             this.state['target'] = findDom;
   //         }
   //     },
   //     parseDom : function(){
   //         var module = this;
   //         var itemTarget = this.state['target'];
   //         if( itemTarget && itemTarget.length > 0 ){
   //             var checkId = [];
   //             var availableItem = this.availableItem(itemTarget);
   //             if( availableItem ){
   //                 $.each( availableItem, function(){
   //                     var id = $(this).attr('id');
   //                     if( id ){checkId.push(id);}
   //                 });
   //                 if( checkId.length > 0 ){
   //                     module.requestApi(checkId);
   //                 }
   //             }
   //         }
   //     },
   //     checkNowDate : function(){
   //         var date = new Date();
   //         var year = date.getFullYear();
   //         var month = '0'+(date.getMonth()+1);
   //         var day = '0'+date.getDate();
   //         var hour = '0'+date.getHours();
   //         var min = '0'+date.getMinutes();
   //         var sec = '0'+date.getSeconds();
   //         month = month.substr(month.length-2, 2);
   //         day = day.substr(day.length-2, 2);
   //         hour = hour.substr(hour.length-2, 2);
   //         min = min.substr(min.length-2, 2);
   //         sec = sec.substr(sec.length-2, 2);
   //         return year+month+day+hour+min;
   //     },
   //     availableItem : function(itemTarget){
   //         /** TODO : dom 처리부분을 위해 생성함... 주석처리 dom 이 필요한 경우 */
   //         var nowDate = parseInt(this.checkNowDate());
   //         var adStop = '<!--광고 중지되었습니다 -->';
   //         var adNotService = '<!--서비스 기간이 아닙니다.-->';
   //         var result = [];
   //         $.each( itemTarget, function(){
   //             var item = $(this);
   //             var useAd = item.attr('usead');
   //             var periodYn = item.attr('periodyn');
   //             var sDate = parseInt(item.attr('sdate')) || null;
   //             var eDate = parseInt(item.attr('edate')) || null;
   //             if( useAd === 'Y' ){
   //                 if( periodYn === 'N'
   //                     || ((sDate && sDate <= nowDate) && (eDate && eDate >= nowDate))
   //                     || ((sDate && sDate <= nowDate) && (!eDate || eDate === 0 ))
   //                 ){
   //                     result.push(this);
   //                 } else {
   //                     item.before(adNotService);
   //                     item.remove();
   //                 }
   //             } else {
   //                 item.before(adStop);
   //                 item.remove();
   //             }
   //         });
   //         return result;
   //     },
   //     requestApi : function(id){
   //         if( typeof this.props.request === 'function' ){
   //             this.props.request(this, id);
   //         }
   //     },
   //     request : function(item, id){
   //         if( typeof this.props.request === 'function' ){
   //             this.props.request(this, item, id);
   //         }
   //     },
   //     init : function(){
   //         this.findComponent();
   //         this.parseDom();
   //     }
   // };
   // ComponentContentCompiler.prototype.constructor = ComponentContentCompiler;
   // window.YNA_SERVICE['MODULE']['CPNTCompiler'] = function(props){ return new ComponentContentCompiler(props); };

    /** [More Content] Create And Apply - 본문 더보기 모듈 */
    MoreContentModule = function(props){
        this.props = props;
        this.isModule = true;
        this.dom = {
            moreButton : null,
            moreButtonCount : null,
            moreBody : null,
            loadingButton : null,
            inputTotalCount : null,
            inputListCount : null,
            inputPageCount : null,
            host : 'en'
        };
        this.state = {
            lang : LANG,
            totalCount : 0,
            toplistCount : 0,
            listCount : 0,
            pageCount : 0,
            defaultList : 0,
            maxCount : 0,
            isKRNk : false
        };
        this.init(this);
    };
    MoreContentModule.prototype = {
        activeDom : function(){
            var props = this.props;
            this.dom.moreButton = $(props.moreButton);
            this.dom.moreBody = $(props.moreListTarget);
            this.dom.moreButtonCount = $(props.moreButtonCount);
            this.dom.loadingButton = $(props.loadingButton);
            this.dom.inputTotalCount = $(props.inputTotalCount);
            this.dom.inputPageCount  = $(props.inputPageCount);
            this.dom.inputListCount  = $(props.inputListCount);
            this.dom.host = $(props.host);
            this.state.toplistCount = $(props.topListTarget).length;
            this.state.defaultList = (this.dom.inputListCount.length === 1)?parseInt(this.dom.inputListCount.val())+this.state.toplistCount:this.dom.moreBody.children().length+this.state.toplistCount;
            this.state.listCount = this.state.defaultList;
            this.state.isKRNk = props.isKRNk;

            if( this.dom.moreButton.length === 0 || this.dom.moreBody.length === 0 ){this.isModule = false;}
        },
        requestData : function(){
            if( typeof this.props.request === 'function' ){
                var target = this.dom.moreBody;
                var totalCount = this.state.totalCount = (this.dom.inputTotalCount.length === 1)?parseInt(this.dom.inputTotalCount.val()):0;
                var pageCount = this.state.pageCount = (this.dom.inputPageCount.length === 1)?parseInt(this.dom.inputPageCount.val()):0;
                var listCount = this.state.listCount || 0;
                typeof this.props.request(this, target, totalCount, pageCount, listCount);
            }
        },
        setMoreContent : function(target, data, viewPage){
            if( target && data ){
                if( typeof this.props.render === 'function' ){
                    this.props.render(target, data, this.state, this.dom);
                    this.setCheckMoreButtonRender();
                }
            } else if(!data && viewPage){
                this.setCheckMoreButtonRender(true);
            }
        },
        setCheckMoreButtonRender : function(hide){
            if( hide == undefined ) hide = false;
            var listCount = parseInt(this.state.listCount);
            var totalCount = parseInt(this.dom.inputTotalCount.val());
            if( listCount >= totalCount || hide ) {
                if( !this.state.isKRNk ) this.dom.moreButton.hide();
                else {
                    this.dom.moreButton.find('span.txt').text('더이상 콘텐츠가 없습니다.');
                    this.dom.moreButton.find('em.num-open').text(listCount);
                    this.dom.moreButton.off('click');
                }
                this.dom.loadingButton.hide();
            }
        },
        activeControl : function(module){
            module.dom.moreButton.on('click', function(){
                module.requestData();
            });
        },
        init : function(module){
            $(function(){
                module.activeDom();
                if( module.isModule ){
                    module.activeControl(module);
                    module.setCheckMoreButtonRender();
                }
            });
        }
    };
    MoreContentModule.prototype.constructor = MoreContentModule;
    window.YNA_SERVICE['MODULE']['MoreContentCompiler'] = function(props){
        return new MoreContentModule(props);
    };
}(window, jQuery));
/** [Compiler Core] Module End ----------------------------------------------------------- */

// 광고 컴포넌트 있을 때 무조건 실행되야함
(function($){
    $.fn.shuffle = function() {
        return this.each(function(){
            var items = $(this).children().clone(true);
            return (items.length) ? $(this).html($.shuffle(items)) : this;
        });
    };
    $.shuffle = function(arr) {
        for(var j, x, i = arr.length; i; j = parseInt(Math.random() * i), x = arr[--i], arr[i] = arr[j], arr[j] = x);
        return arr;
    };
    $(function(){
        $('.yna-ad-shuffle').shuffle();
    });
})(jQuery);

/** [Compiler Control] Module Start ------------------------------------------------------ */
$(document).ready(function(){
    /** UI Module */
    var UI = window.YNA_SERVICE['YNA_UI'];
    var YNA_MODULE = window.YNA_SERVICE['MODULE'];
    var YNA_ViewCheck = UI.isViewPage();
    var YNA_SectionValue = UI.isSectionValue();
    var YNA_UrlObject = UI.getUrlObject();
    var YNA_CheckDevice = UI.checkDevice();
    // var YNA_UserProps = (function(){
    //     var result = {'blockAd' : true, 'subTitle' : ''};
    //     if( YNA_ViewCheck ){
    //         var url = encodeURI(APS+'content?id='+YNA_UrlObject.cid);
    //         $.ajax({
    //             url : url,
    //             method : 'GET',
    //             async: false,
    //             cache : true,
    //             success : function(param){
    //                 var data = param['DATA'][0];
    //                 var url = "";
    //
    //                 if( data ){
    //                     if( data['AD_YN'] === 'Y' ){
    //                         result['blockAd'] = false;
    //                     }
    //                     result['subTitle'] = data['SUBTITLE'] || '';
    //                 } else {
    //                     result['blockAd'] = false;
    //                 }
    //                 /** 컨텐츠 높이 계산 */
    //                 window.YNA_SERVICE['RESIZE']();
    //             },
    //             fail : function(){
    //                 //console.log('[error] Ajax Error - Check Please Ajax Url Or Data!!');
    //             }
    //         });
    //     } else {
    //         result['blockAd'] = false;
    //     }
    //     return result;
    // }());
    
    /** Advertisement Compiler Start */
    if( !YNA_UrlObject.domain1 || !YNA_UrlObject.cid ){
        var newUrl = $('head').find('meta[name=url]').attr('content');
        YNA_UrlObject = UI.getUrlObject(newUrl);
    }
   // if( YNA_UrlObject.domain1 && YNA_UrlObject.cid ){
   //     if( YNA_UrlObject.domain1 === 'cb' ){ YNA_UrlObject.domain1 = 'cn' }
   //     var ajaxUrl = APS+'svc.v.ad?host='+YNA_UrlObject.domain1+'&section='+YNA_UrlObject.section+'&cid='+YNA_UrlObject.cid;
   //     ajaxUrl = encodeURI(ajaxUrl);
   //     if( !YNA_UserProps.blockAd ){
   //         YNA_MODULE.ADCompiler({
   //             ajaxTargetUrl     : ajaxUrl,
   //             paragraphClass    : 'yna-p',
   //             ajaxMethod        : 'GET',
   //             ajaxDataType      : 'json',
   //             texTypeName       : 'yna_component',
   //             isFadeIn          : true
   //         });
   //     }
   // }

    /** 본문 보기 영역 가동 모듈 */
    if( YNA_ViewCheck ){
        var viewCid = YNA_UrlObject['cid'];
        
        /** 다른 언어의 기사일 시 페이지 이동 */
        if( LANG_TYPE.toUpperCase() != viewCid.slice(1,3) && /A|X/i.test(viewCid.slice(0,1)) ){
        	location.href = "http://yna.kr/" + YNA_UrlObject["cid"];
        }
         
        /** 북마크 데이터 로컬 스토리지에 저장 */
        if( LANG_TYPE != 'kr' ){
        	window.CONTENT_DATA?(function(){
        		localStorage.setItem("bookMarkData", window.CONTENT_DATA);
        	})():(function(){
        		var url = encodeURI(APS+'content?id='+viewCid);
                $.ajax({
                    url : url,
                    method : 'GET',
                    async: false,
                    cache : true,
                    success : function(param){
                    	if( param == null || param['DATA'].length < 1 ) {
                            return false;
                        }
                    	localStorage.setItem("bookMarkData", JSON.stringify(param['DATA'][0]));
                    }
                });
        	})();
        }
        
        /** 본문용 구글광고 위치 변경 */
        var ynaParagraph = $('.article').children().not('.adrs');
        var findAdGoogle = $('.article-ad-box');
        var isAppendNum = 4;
        
        if( viewCid.substr(0,1).toLowerCase() !== 'i' ){        	        	
        	var ynaDomainId = $('#domainId').val();
        	/** m-kr renewal  **/
        	if(ynaDomainId=="2102")        	
        	{
                //타입별 단락 리스트
                var $textList = $("div.article > p"); //문자열 단락
                var $refList = $("div.article").children().not(".tit-sub");  //소재 포함 단락, 부제목 제외

                //광고 박스 리스트
                $("aside.article-boxad").each(function (idx, box) {
                    //현재 광고 박스
                    var $box = $(box);

                    //설정 정보 확인
                    var position = $box.data("position") || -1;                                   //정수. 삽입 단락 위치. (0:맨앞 1:1단란뒤... N:N단락뒤) (필수)
                    var minimum = $box.data("minimum") || -1;                                     //정수. 광고 삽입할 최소 단락 수 (필수)
                    var includeYna = $box.data("includeyna") == false ? false : true;     //true/false. 소재 포함 단락 카운트 여부 기본 true)
                    var force = $box.data("force") == true ? true : false;                  //true/false. position 보다 단락이 적을 경우 삽입할지 여부 (기본 false)

                    //소재 포함 여부에 따른 단락 리스트
                    var $list = includeYna ? $refList : $textList;
                    var pnum = $list.length; //단락 개수

                    if (_param.test == "true" && console.table && console.log) {
                        console.log("$list", $list);
                        console.table([
                            ["pnum", pnum],
                            ["position", position],
                            ["minimum", minimum],
                            ["includeYna", includeYna],
                            ["force", force]
                        ]);
                    }

                    //position 존재 해야함 (필수)
                    //단락 개수가 position 보다 클고 최소 단락개수보다 큰경우 해당 단락에 삽입
                    if ( position >= 0 && pnum >= position && pnum >= minimum ) {
                        $list.eq(position).before($box);
                    }
                    //단락 개수가 position 보다 작지만 force 가 true인 경우 마지막 단락 뒤에 광고를 삽입
                    else if (position >= 0 && force == true) {
                        $list.last().after($box);
                    }
                    //최소 단락을 갖추지 못한경우 해당 광고 삭제
                    else {
                        $box.remove();
                    }
                });
        		
        		/*
        		var cid = UI.getArticleId();
                isAppendNum = 1;
                findAdGoogle = $('.article-boxad');

                ynaParagraph = $('.article p');
                var ynaParagraphBody = [];

                $.each(ynaParagraph, function(idx){
                    var text =  $.trim($(this).text());
                    if( text.length > 0 ) ynaParagraphBody.push($(this));
                });

                if( findAdGoogle && findAdGoogle.length > 0 ){
                    findAdGoogle = findAdGoogle.eq(0);
                    if( ynaParagraphBody.length >= 2 ){
                        $(ynaParagraphBody[isAppendNum]).after(findAdGoogle);
                        window.YNA_SERVICE['RESIZE']();
                    }else{
                        findAdGoogle.remove();
                    }
                }else{
                    findAdGoogle.remove();
                }
                */
        	}
        	else
        	{
        		/** 국문PC */
	            if( location.host.match('www.yna') ) {
	            	console.log('media compiler work');
	                var chkNext1, chkNext2;
	                if( findAdGoogle && findAdGoogle.length > 0 ){
	                    findAdGoogle = findAdGoogle.eq(0);
	                    if( ynaParagraph.length >= 9 ){
	                    	console.log('media p true');
	                        var pParagraph = $('.article').children('p').not('.adrs, .youtu');
	                        var endCount = pParagraph.length-2;
	                        for( var i = isAppendNum; i <= endCount; i++ ) {
	                            var currentParagraph = pParagraph.eq(i);
	                            var nextParagraph1 = currentParagraph.next();
	                            var nextParagraph2 = nextParagraph1.next();
	                            nextParagraph1[0].tagName == 'P' ?  chkNext1 = true : chkNext1 = false;
	                            nextParagraph2[0].tagName == 'P' ?  chkNext2 = true : chkNext2 = false;
	                            console.log('media c true');
	                            if( chkNext1 && chkNext2 ){
	                                if( pParagraph[i].innerText ){
	                                    $(pParagraph[i]).before(findAdGoogle);
	                                	console.log('media move'); 
	                                    window.YNA_SERVICE['RESIZE']();
	                                    break;
	                                }
	                            }else{
	                                if( i == endCount ) findAdGoogle.remove();
	                            }
	                        }
	                        if( isAppendNum > endCount ) findAdGoogle.remove();
	                    }else{
	                    	console.log('media p false');
	                        findAdGoogle.remove();
	                    }
	                }
	            }
	            /** 국문모바일 2단락 뒤 */
	            else if( location.host.match('m.yna') || location.host.match('m-kr'))
	            {
	                var cid = UI.getArticleId();
	                isAppendNum = 1;
	                findAdGoogle = $('.article-boxad');
	
	                if( !cid.match('MYH') ) {
	                    ynaParagraph = $('div.section-txt p');
	                    var ynaParagraphBody = [];
	
	                    $.each(ynaParagraph, function(idx){
	                        var text =  $.trim($(this).text());
	                        if( text.length > 0 && !$(this).hasClass('sttl') ) ynaParagraphBody.push($(this));
	                    });
	
	                    if( findAdGoogle && findAdGoogle.length > 0 ){
	                        findAdGoogle = findAdGoogle.eq(0);
	                        if( ynaParagraphBody.length >= 2 ){
	                            $(ynaParagraphBody[isAppendNum]).after(findAdGoogle);
	                            window.YNA_SERVICE['RESIZE']();
	                        }else{
	                            findAdGoogle.remove();
	                        }
	                    }else{
	                        findAdGoogle.remove();
	                    }
	                }
	                else {  // 영상본문일 경우 광고 처리 추가
	                    isAppendNum = 1;
	                    ynaParagraph = $('.dev_body').html();
	                    var ynaParagraphBody = [];
	
	                    if( ynaParagraph.match('<br>') ){
	                        ynaParagraph = ynaParagraph.split('<br>');
	                        $.each(ynaParagraph, function(idx){
	                            var text = $.trim(this);
	                            if( text.length > 0 ) ynaParagraphBody.push(this);
	                        });
	                    }
	                    
	                    if( findAdGoogle && findAdGoogle.length > 0 ){
	                        if( ynaParagraphBody.length >= 2 ){
	                            $.each(ynaParagraph, function(idx){
	                                if( ynaParagraph[idx] == ynaParagraphBody[isAppendNum] ) {
	                                    //ynaParagraph.splice(idx, 0, '<span class="article-boxad">'+findAdGoogle.html()+'</span>');
	                                    ynaParagraph[idx] = '<span class="article-boxad">'+findAdGoogle.html()+'</span>'+ynaParagraph[idx];
	                                    return false;
	                                }
	                            });
	                        }
	                        var adHtml = ynaParagraph.join('<br>');
	                        $('.dev_body').html(adHtml);
	                        findAdGoogle.remove();
	                    }else{
	                        findAdGoogle.remove();
	                    }
	                }
	            }            
	            /** 다국어모바일 2단락 뒤 */
	            else if( location.href.match('m-') ){
	                ynaParagraph = $('.text-group');
	                isAppendNum = 1;
	
	                if( findAdGoogle && findAdGoogle.length > 0 ){
	                    findAdGoogle = findAdGoogle.eq(0);
	                    if( ynaParagraph.length >= 2 ){
	                        $(ynaParagraph[isAppendNum]).after(findAdGoogle);
	                        window.YNA_SERVICE['RESIZE']();
	                    }else{
	                        findAdGoogle.remove();
	                    }
	                }else{
	                    findAdGoogle.remove();
	                }
	            }
        	}
        }
        /** 본문용 구글광고 위치 변경 */

        /** Multi Group(Photo+Video) Slide Content Create And Compiler Start */
        YNA_MODULE.MultiGroupCompiler({
            isModuleActive : true,
            /** 기본 사용자 Props */
            customTagName : 'div',
            groupClass    : '.yna_group_slide',
            videoCodeName : 'mpic',
            videoTitle    : 'Videos',
            imageCodeName : 'image',
            imageTitle    : 'Photos',
            nextBtnClass  : '.btn-next02',
            prevBtnClass  : '.btn-prev02',
            mediaUrl      : YNA_MODULE['ynaCdnUrl'],
            isFadeIn      : true
        });

        /** Photo Group Slide Content Create And Compiler Start */
        YNA_MODULE.PhotoGroupCompiler({
            isModuleActive : true,
            /** 기본 사용자 Props */
            deviceType    : YNA_CheckDevice,
            customTagName : 'div',
            groupClass    : '.yna-img-slide',
            imageCodeName : 'image',
            imageTitle    : 'Photos',
            prevBtnClass  : '.btn-prev02',
            nextBtnClass  : '.btn-next02',
            isFadeIn      : true
        });

        /** Video Group Slide Content Create And Compiler Start */
        YNA_MODULE.VideoGroupCompiler({
            isModuleActive : true,
            /** 기본 사용자 Props */
            deviceType    : YNA_CheckDevice,
            customTagName : 'div',
            groupClass    : '.yna-mpic-slide',
            videoCodeName : 'mpic',
            videoTitle    : 'Videos',
            mediaUrl      : YNA_MODULE['ynaCdnUrl'],
            isFadeIn      : true
        });

        /** Social Group Content Create And Compiler Start */
        YNA_MODULE.SocialGroupCompiler({
            isModuleActive : true,
            /** 기본 사용자 Props */
            deviceType    : YNA_CheckDevice,
            customTagName : 'div',
            groupClass    : '.yna-sns',
            totalFacebook  : $('.yna-sns[data-source="facebook"]').length,
            totalTwitter    : $('.yna-sns[data-source="twitter"]').length,
            totalInstagram   : $('.yna-sns[data-source="instagram"]').length
        });

        /** Hyper Group Content Create And Compiler Start */
        YNA_MODULE.HyperGroupCompiler({
            isModuleActive : true,
            /** 기본 사용자 Props */
            customTagName : 'span',
            groupClass    : '.yna-hyper',
            externalLink   : true
        });

       /** 광고형 또는 기간형 컴포넌트 컴파일러 Start */
       // YNA_MODULE.CPNTCompiler({
       //     className     : '.yna_reload',
       //     customTagName : 'yna',
       //     request : function(module, id){
       //         if( !YNA_UserProps.blockAd ){
       //             var control = this;
       //             var urlData = (UI.getUrlQuery())['urlData'];
       //             var query = (UI.getUrlQuery())['query'];
       //             var host = urlData['host'].split('.')[0];
       //             var checkUrl = UI.getUrlObject();
       //             var idString = id.join(',');
       //             if( host === 'cb' ){ host = 'cn'; }
       //             var url = APS+'svc.merge?host='+host+'&section='+checkUrl.section+'&ids='+idString;
       //             url = encodeURI(url);
       //             $.ajax({
       //                 url : url,
       //                 method : 'GET',
       //                 async: false,
       //                 cache : true,
       //                 success : function(data){
       //                     if( data && data['DATA'] ){
       //                         control.render(data['DATA']);
       //                         /** 컨텐츠 높이 계산 */
       //                         window.YNA_SERVICE['RESIZE']();
       //                     }
       //                 },
       //                 fail : function(){
       //                     //console.log('[error] Ajax Error - Check Please Ajax Url Or Data!!');
       //                 }
       //             });
       //         }
       //     },
       //     render : function(param){
       //         var control = this;
       //         /** script string 을 체크하기 위한 메소드입니다. */
       //         var checkScriptString = function(string){
       //             var re = /<script\b[^>]*>([\s\S]*?)<\/script>/gm;
       //             var math = re.exec(string);
       //             return (!math || !math.length)?false:math.length > 0;
       //         };
       //         /** 광고 파라미터 데이터 유무 확인 */
       //         if( param.length > 0 ){
       //             $.each(param, function(num, state){
       //                 var dataId = state['COMPONENT_ID'];
       //                 var findStr = control['customTagName']+'#'+dataId+control['className'];
       //                 var target = $('body').find(findStr);
       //                 if( target.length > 0 ){
       //                     if( state['BODY'] ){
       //                         /** TODO : script string 에러에 대한 예외 처리 */
       //                         try {
       //                             target.before(state['BODY']);
       //                         } catch (err) {
       //                             console.log(err, state['BODY']);
       //                         }
       //                         /** TODO : script string 체크를 통해 아래처럼 해도 되지만...
       //                          * jQuery에서 자동으로 script eval 처리 스위칭이 된다고 생각되기 때문에 주석 처리합니다... */
       //
       //                         /**
       //                          //console.log(state['BODY'])
       //                          //console.log(checkScriptString(state['BODY']))
       //                          if( checkScriptString(state['BODY']) ){
       //                         // string eval error 가 없을경우 아래를... 사용해도 됩니다~
       //                         target.before(state['BODY']);
       //                     } else {
       //                         // eval error 대응하기 위한 소스입니다.. 필요할 수 있으니 지우지 마세요..
       //                         var newDom = document.createElement('span');
       //                         target.before(newDom);
       //                         newDom.outerHTML = state['BODY'];
       //                     }
       //                          */
       //                     }
       //                     target.remove();
       //                 }
       //             });
       //         }
       //     }
       // });

        /** Photo IPT Slide Content Create And Compiler Start */
        YNA_MODULE.PhotoIPTCompiler({
            domainParse   : (setImgDomain)?setImgDomain:null, //이건 도메인 붙여주는 모들을 넣어야함...
            groupClass    : '.yna-ipt-slide',
            customTagName : 'div',
            imageCodeName : 'image',
            imageTitle    : 'Photos',
            prevBtnClass  : '.btn-prev02',
            nextBtnClass  : '.btn-next02',
            urlObject      : UI.getUrlQuery()['urlData'],
            isFadeIn      : true,
            request       : function(module, item, data){
                var url = APS+'issue.contents?id='+data['data-refid']+'&total=Y';
                url = encodeURI(url);
                $.ajax({
                    url : url,
                    method : 'GET',
                    async: false,
                    cache : true,
                    success : function(data){
                        module.setIptContent(item, data['DATA']);
                        /** 컨텐츠 높이 계산 */
                        window.YNA_SERVICE['RESIZE']();
                    },
                    fail : function(){
                        //console.log('[error] Ajax Error - Check Please Ajax Url Or Data!!');
                    }
                });
            }
        });
    }

    /** 더보기 컨텐츠에 대한 컴파일러 Start */
    YNA_MODULE.MoreContentCompiler({
        moreButton      : '.yna-more-count',
        moreButtonCount : '.yna-more-count',
        moreListTarget  : '.yna-more',
        loadingButton   : '.yna-more-load',
        topListTarget   : '.yna-more-top li',
        inputTotalCount : 'input#totalCount',
        inputPageCount  : 'input#page',
        inputListCount  : 'input#count',
        host : UI.getUrlQuery()['urlData']['host'].split('.')[0],
        isKRNk : LANG_TYPE == 'kr' && UI.getUrlQuery()['urlData']['path'].match('nk') ? true : false,
        requestCp : function(host, section){
            /** 테스트용 */
            var result;
            $.ajax({
                url : encodeURI(APS+'findcp?host='+host+'&section='+section),
                method : 'GET', async: false, cache : true,
                /** 테스트를 못해서 data 의 타입을 확실히 모르겠음.. 일단 string 이라고 가정하고... */
                success : function(data){ result = data; },
                fail : function(){ console.info('findcp Error!')}
            });
            return result;
        },
        request : function(module, target, totalCount, pageCount, listCount){
            module.dom.loadingButton.show();
            var urlData = (UI.getUrlQuery())['urlData'];
            var host = urlData['host'].split('.')[0];
            var section = YNA_SectionValue;
            var nextPage = parseInt(pageCount)+1;
            var isViewPage = YNA_ViewCheck;
            var newUrl = urlData['href']; //http://dev-cn.yna.co.kr/image/gallery?page=2&cp=paging
            if( newUrl.indexOf('#') ) newUrl = newUrl.split('#')[0];
            var url = '';
            if( newUrl.indexOf('?') > 0 )
                url = newUrl+'&page='+nextPage+'&cp=paging';
            else
                url = newUrl+'?page='+nextPage+'&cp=paging';
            if( isViewPage ){ url = url+'&view=Y'; }
            $.ajax({
                url : encodeURI(url),
                method : 'GET',
                async: false,
                cache : true,
                success : function(data){
                    module.setMoreContent(target, data, isViewPage);
                    /** 컨텐츠 높이 계산 */
                    window.YNA_SERVICE['RESIZE']();
                }
            });
        },
        render : function(target, data, state, dom){
            if( data ){
                var dummyDom = $(data);
                if( dummyDom.length > 0 ){
                    //  festivals/all만 예외처리(.yna-more section 구조)
                    if( !location.href.match('festivals/all') )
                        var item = dummyDom.find('li');
                    else
                        var item = dummyDom.find('section');
                    var currentPage = state.pageCount+1;
                    var currentCount = state.defaultList*currentPage;
                    target.append(item);
                    if( item.find('span').hasClass('datefm-'+state.lang.toLowerCase()+'01') ) {
                        changeDateFormat.init();
                    }
                    $.each(item.find('.btn-like'), function(idx, btn){
                        if( $(btn).attr('data-likeid') ){
                            var likeOnChk = like.check($(btn).attr('data-likeid'));
                            if( likeOnChk ) $(btn).addClass('btnlike-on');
                        }
                    });
                    dom.inputTotalCount.val(state.totalCount);
                    dom.inputPageCount.val(currentPage);
                    state.listCount = state.listCount+item.length;
                    dom.moreButtonCount.find('em').text(state.listCount);
                    if( item.find('img').length ) {
                        var imgTot = 0;
                        $.each(item.find('img'), function(){
                           if( $(this).attr('src').length > 22 ) imgTot++;
                        });
                        if( imgTot == 0 ) dom.loadingButton.hide();
                        else {
                            var cnt = 0;
                            item.find('img').on('load error', function(){
                                cnt++;
                                if( cnt == imgTot  ) dom.loadingButton.hide();
                            });
                        }
                    } else dom.loadingButton.hide();
                    try{
                        ( imgCrop ? imgCrop() : '' );
                        ( like.setEvent ? like.setEvent() : '');
                    }catch(e){}
                }
            }
        }
    });

    /** 중국어 CB 문자 오류 처리 : 20170715 */
    (function chinaHrefStringChange(){
        var urlData = (UI.getUrlQuery())['urlData'];
        var host = urlData['host'].split('.')[0];
        if( host === 'cb' ){
            setTimeout(function(){
                var targetLink = '//cb.yna.co.kr';
                var targetString = '//cb';
                var allALink = $('body a');
                var titALink = $('.section-major-news .tit a, .section-major-news .btn-more a');
                $.each(allALink, function(){
                    var item = $(this);
                    var href = item.attr('href');
                    try{
                        var array = href.split('/view/');
                        var first = array[0];
                        if( !first || href.indexOf('/view') === 0 ){
                            item.attr('href' , 'view/'+array[1]);
                        }
                    }catch(e){

                    }
                });
                $.each(titALink, function(){
                    var item = $(this);
                    var href = item.attr('href');
                    if( href.indexOf(targetString) === 0 ){
                        if(  href.substr(0, 14) === targetLink ){
                            href = href.substr(15,href.length-1);
                            item.attr('href' , href);
                        }
                    }
                });
            }, 1000);
        }
    }());

    /** XED 인물 삽입기사의 이미지 로딩시 이미지 src 로딩 실패 시 돔 처리 : 20180702 */
    (function xedPeopleImageCheckParse(){
        var personZone = $('.person-zone');
        if( personZone.length > 0 ){
            $.each(personZone, function(){
                var hasImg = $(this).find('img');
                if( hasImg.length > 0 ){
                    hasImg.on("error", function(){hasImg.hide();});
                }
            });
        }
    }());
});
