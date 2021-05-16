var YNAMerger = {
	list : [],
	load : function()
	{
		var comp = document.querySelectorAll('div[data-cp]');
		var cpid;
		for (var i=0; i<comp.length; i++) {
			cpid = comp[i].getAttribute('data-cp');
			if (!YNAMerger.list.indexOf(cpid) >= 0)
			{
				YNAMerger.list.push(cpid);
				YNAMerger.merge(cpid);
			}
		}
	},
	invoke : function(cp)
	{
		try
		{
			if (cp)
			{
				if (typeof cp == 'function')
					cp();
				else if (typeof cp == 'string')
					eval(cp);
				else if (typeof cp == 'object') 
				{
					if (isNaN(cp.length) === false)
					{
						for (var i=0; i<cp.length; i++)
						{
							YNAMerger.invoke(cp[i]);
						}
					}
				}
				else
				{
					console.log('CP_Callback is wrong type. must be [function] or [string]');
				}
			}
		}catch(e)
		{
			console.log('CP_Callback invoke error');
			console.log(e);
		}
	},
	addCallback: function(cb)
	{
		if (window.CP_Callback)
		{
			var cp = window.CP_Callback;
			if (typeof cp == 'object' && isNaN(cp.length) === false)
			{
				cp.push(cb);
			}
			else
			{
				var list = [];
				list.push(cp);
				list.push(cb);
				window.CP_Callback = list;
			}
		}
		else
		{
			window.CP_Callback = cb;
		}
	},
	invokeCallback: function()
	{
		YNAMerger.invoke(window.CP_Callback);
	},
	getXHR : function()
	{
		if (window['XMLHttpRequest'])
		{
			return new XMLHttpRequest();
		}
		else	// for old ID
		{
			var list = [
				"MSXML2.XmlHttp.6.0",
		        "MSXML2.XmlHttp.5.0",
		        "MSXML2.XmlHttp.4.0",
		        "MSXML2.XmlHttp.3.0",
		        "MSXML2.XmlHttp.2.0"
		        ];
			for (var i=0; i<list.length; i++)
			{
				try
				{
					var xhr = new ActiveXObject(list[i]);
					return xhr;
				}catch(e){}
			}
			return null;
		}
	},
	getList : function(id)
	{
		return document.querySelectorAll('div[data-cp="' + id + '"]')
	}
	,
	merge : function(id)
	{
		if (!id || !/^[0-9]{1,6}$/.test(id))
		{
			//console.log('wrong cp [' + id + ']');
			return;
		}
		var xhr = YNAMerger.getXHR();
		if (!xhr)
		{
			var div = YNAMerger.getList(id);
		}
		var path = '';
		var loc = document.location;
		// for merge
		if (loc.search && loc.search.indexOf("?url=") > -1)
		{
			var pl = loc.search.split('&');
			var param = pl[0].replace(/%3[fF]/g,'?');
			var n = param.indexOf('?',1);
			if (n > 0)
				param = param.substring(0,n);
			path = loc.pathname + param + '%3Fcp=' + id;
		}
		else
		{	
			path = loc.pathname + '?cp=' + id;
		}
		
		xhr.open('GET', path, true);
		xhr.onreadystatechange = function (oEvent) {  
		    if (xhr.readyState === 4) {  
		        if (xhr.status === 200) {
		        	var div = YNAMerger.getList(id);
					var result = xhr.responseText.replace(/^\s*/,'');
					if (xhr.status == 200)
					{
						for (var i=0; i<div.length; i++) {
							YNAMerger.setHTML(div[i], result, id);
						}
					}
					else
					{
						for (var i=0; i<div.length; i++) {
							div[i].innerHTML = result;
							div[i].setAttribute('data-cp-' + id, 'false');
						}
					}
					YNAMerger.remove(id);  
		        } else {  
		           console.log("Error", xhr.statusText);  
		        }  
		    }  
		}; 
		xhr.send();
	},
	remove: function(id)
	{
		for (var i=0; i<YNAMerger.list.length; i++)
		{
			if (YNAMerger.list[i] == id)
			{
				//console.log("delete : " + id);
				delete YNAMerger.list[i];
				break;
			}
		}
		if (YNAMerger.list.join('') == '')
		{
			console.log('load all! invoke CP_Callback');
			YNAMerger.invokeCallback();
		}
	},
	setHTML: function(ele, html, id)
	{
		if (html && !/<html[^>]*>/.test(html))
		{	
			ele.outerHTML = html;
			ele.setAttribute('data-cp', id);
			var sl = ele.querySelectorAll('script');
			for (var i=0; i<sl.length; i++)
			{
				var s = document.createElement('script');			
				s.text = sl[i].innerHTML;
				var attr = sl[i].attributes;
				for (var j=0; j<attr.length; j++)
				{
					s.setAttribute(attr[j].name, attr[j].value);
				}
				sl[i].parentNode.replaceChild(s, sl[i]);
			}
		}
		else
		{
			console.log('cannot set HTML Element : CP-' + id);
		}
	}
}
YNAMerger.load();