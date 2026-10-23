---
layout: page
title: 訂閱
permalink: /subscribe/
---

每週一封，一篇實測，不寄廣告。

{% if site.buttondown_user != "" %}
<form action="https://buttondown.com/api/emails/embed-subscribe/{{ site.buttondown_user }}" method="post" target="popupwindow" style="margin:1em 0">
  <label for="bd-email">Email</label>
  <input type="email" name="email" id="bd-email" required style="padding:6px;min-width:240px">
  <input type="submit" value="訂閱" style="padding:6px 14px">
</form>
{% else %}
電子報即將開放。現在可以先用 [RSS]({{ "/feed.xml" | relative_url }}) 追蹤。
{% endif %}
