if (document.querySelector) {
  var intro = document.querySelector('.site-intro');
  var nav = document.querySelector('.site-nav');
  var menu = document.querySelector('.site-menu');
  var introHeight = intro.offsetHeight;

  if (window.getComputedStyle &&
      getComputedStyle(nav).position === 'relative') { // sticky not supported
    nav.style.height = nav.offsetHeight + 'px';
    var isMenuFixed = false;
  }
  onscroll = function() {
    fixMenu();

    if (window.URL) {
      var allposts = document.querySelector('.wrap:first-child .all-posts');
      if (!allposts) {
        return;
      }
      var wraps = document.querySelectorAll('.page .wrap');
      if (!wraps.length) {
        return;
      }
      var link = allposts.querySelectorAll('a')[
        Array.prototype.reduce.call(wraps, function(carry, wrap, i) {
          return wrap.getBoundingClientRect().top < 0 ? i : carry;
        }, 0)
      ];
      if (link.href !== location.href) {
        history.replaceState({}, link.textContent, link.href);
      }
    }

    if (shouldRestoreScrollPosition) {
      storeScrollPosition();
    }
  };

  onresize = function() {
    (window.requestAnimationFrame || setTimeout)(function() {
      introHeight = intro.offsetHeight;
      fixMenu();

      if (window.URL) {
        positionUnderline();
      }
    });
  };
}

function fixMenu() {
  if (isMenuFixed === undefined) {
    return;
  }

  if (scrollY >= introHeight) {
    if (!isMenuFixed) {
      isMenuFixed = true;
      menu.classList.add('site-menu-fixed');
    }
  } else {
    if (isMenuFixed) {
      isMenuFixed = false;
      menu.classList.remove('site-menu-fixed');
    }
  }
}

if (window.URL) {
  onpopstate = onBack;
  onclick = onClick;
  var shouldRestoreScrollPosition = !safari && ('scrollRestoration' in history);
  shouldRestoreScrollPosition = false;
  history.scrollRestoration = 'manual';

  var classNames = Array.prototype.map.call(
    document.querySelectorAll('.site-menu-item'),
    function(item) { return item.className.replace('site-menu-item', 'site'); }
  );
  var rects = [];

  var ua = navigator.userAgent;
  var safari = /Safari/.test(ua) && !/Chrome/.test(ua) && !/Mobile/.test(ua);
  var ios = /Safari/.test(ua) && !/Chrome/.test(ua) && /Mobile/.test(ua);

  document.documentElement.classList.add('loading');
  onload = function() {
    onload = null;

    document.documentElement.classList.remove('loading');
    document.documentElement.classList.add('loaded');
    setTimeout(function() {
      document.documentElement.classList.remove('loaded');
    }, 900);

    menu.classList.add('site-menu-animated');
    positionUnderline();
    infiniteScroll(document.querySelector('.page'));
  };
}

var timer = 0;
function storeScrollPosition() {
  clearTimeout(timer);
  timer = setTimeout(function() {
    var offset = Math.max(scrollY - introHeight, 0);
    history.replaceState({offset: offset}, document.title, location.href);
  }, 1000);
}

function onBack(event) {
  fetchPage(location.href, false, event.state);
}

function onClick(event) {
  if (event.button || event.ctrlKey || event.altKey || event.metaKey) {
    return;
  }
  for (var node = event.target; node; node = node.parentNode) {
    if (!node.href) {
      continue; // not a link
    }
    var url = (new URL(node.href));
    if (url.host !== location.host) {
      return; // external link
    }
    if (url.pathname === location.pathname) {
      return false; // same page
    }
    if (document.querySelectorAll('.page').length === 1) {
      fetchPage(node.href, true);
    }
    return false; // internal link
  }
}

var cache = {};
function fetchPage(url, shouldPushState, state) {
  var allposts = document.querySelector('.wrap:first-child .all-posts');
  if (allposts) {
    allposts.parentNode.removeChild(allposts);
  }

  if (cache[url]) {
    animatePageTransition(cache[url], shouldPushState && url, state);
    return;
  }

  var req = new XMLHttpRequest();
  req.onerror = function() {
    location.href = url;
    return;
  };
  req.onload = function() {
    if (this.status !== 200) {
      location.href = url;
      return;
    }
    cache[url] = this.responseText;
    animatePageTransition(cache[url], shouldPushState && url, state);
  };
  req.open('get', url);
  req.send();
}

function animatePageTransition(html, url, state) {
  var doc = document.implementation.createHTMLDocument('');
  doc.documentElement.innerHTML = html;

  var offset = Math.max(scrollY - introHeight, 0);
  var main = document.querySelector('.site-content');
  var curr = document.querySelector('.page');

  if (safari && (!url || offset)) {
    if (url) {
      history.pushState({}, doc.title, url);
    }
    if (offset) {
      scrollBy(0, -offset);
    }
    var next = document.importNode(doc.querySelector('.page'), true);
    main.appendChild(next);
    infiniteScroll(next);
    main.removeChild(curr);
    document.title = doc.title;
    document.body.className = doc.body.className;
    if (url) {
      animateUnderline(classNames.indexOf(document.body.className));
    } else {
      positionUnderline();
    }
    return;
  }

  if (state && state.offset) {
    offset -= state.offset;
  }
  if (offset) {
    curr.style.top = -offset + 'px';
    if (safari) {
      curr.style.opacity = 0;
    }
  }
  (safari ? requestAnimationFrame : function(fn) { fn(); })(function() {
    if (offset) {
      scrollBy(0, -offset);
      if (safari) {
        curr.style.opacity = '';
      }
    }

    if (url) {
      history.pushState({}, doc.title, url);
    }
    document.title = doc.title;

    // append requested page
    var next = document.importNode(doc.querySelector('.page'), true);
    main.appendChild(next);
    infiniteScroll(next);

    var currClass = document.body.className;
    var nextClass = doc.body.className;
    var same = currClass === nextClass;
    if (same) {
      curr.classList.add('page-fade');
    } else {
      var currIndex = classNames.indexOf(currClass);
      var nextIndex = classNames.indexOf(nextClass);
      var ltr = currIndex < nextIndex;
      curr.classList.add(ltr ? 'page-exit-left' : 'page-exit-right');
      document.body.className = nextClass;
      animateUnderline(nextIndex);
    }
    if (ios || safari) {
      document.body.clientHeight;
    }

    // remove previous page
    setTimeout(function() {
      main.removeChild(curr);
    }, 600);
  });
}

function positionUnderline() {
  var links = document.querySelectorAll('.site-menu-item a');
  rects = Array.prototype.map.call(links, function(link) {
    return link.getBoundingClientRect();
  });
  var underline = animateUnderline(classNames.indexOf(document.body.className));
  underline.classList.add('positioning');
  underline.clientHeight;
  underline.classList.remove('positioning');
}

function animateUnderline(index) {
  var underline = document.querySelector('.site-menu-underline');
  var rect = rects[index];
  underline.style.left = rect.left + 'px';
  underline.style.width = rect.width + 'px';
  return underline;
}

function infiniteScroll(page) {
  var allposts = page.querySelector('.wrap:first-child .all-posts');
  if (!allposts) {
    return;
  }
  var here = location.href;
  setTimeout(function() {
    if (here != location.href) {
      return;
    }
    Array.prototype.map.call(
      allposts.querySelectorAll('a:not(:first-child)'),
      function(link) { return link.href; }
    ).forEach(function(url, i, urls) {
      if (cache[url]) {
        appendPosts(page, urls);
        return;
      }

      var req = new XMLHttpRequest();
      req.onload = function() {
        if (this.status !== 200) {
          return;
        }
        cache[url] = this.responseText;

        if (here != location.href) {
          return;
        }
        appendPosts(page, urls);
      };
      req.open('get', url);
      req.send();
    });
  }, 3000);
}

function appendPosts(page, urls) {
  var allposts = page.querySelector('.wrap:first-child .all-posts');
  if (!allposts) {
    return;
  }
  var wraps = page.querySelectorAll('.wrap:not(:first-child)');
  urls.slice(wraps.length).every(function(url) {
    var html = cache[url];
    if (!html) {
      return false;
    }
    var doc = document.implementation.createHTMLDocument('');
    doc.documentElement.innerHTML = html;
    var wrap = document.importNode(doc.querySelector('.wrap'), true);
    page.appendChild(wrap);
    return true;
  });
}