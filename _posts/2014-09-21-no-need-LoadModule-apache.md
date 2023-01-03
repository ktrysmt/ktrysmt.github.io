---
layout: post
title: "だいたいコメントアウトするapacheモジュールたち"
date: 2014-09-20 15:09:08 +0900
comments: true
categories: Apache
published: true
---


そもそもApacheをあまり使わなくなってきているが

```
vim /etc/httpd/conf/httpd.conf
```

```sh
 154 #
 155 LoadModule auth_basic_module modules/mod_auth_basic.so
 156 LoadModule auth_digest_module modules/mod_auth_digest.so
 157 LoadModule authn_file_module modules/mod_authn_file.so
 158 #LoadModule authn_alias_module modules/mod_authn_alias.so
 159 #LoadModule authn_anon_module modules/mod_authn_anon.so
 160 #LoadModule authn_dbm_module modules/mod_authn_dbm.so
 161 LoadModule authn_default_module modules/mod_authn_default.so
 162 LoadModule authz_host_module modules/mod_authz_host.so
 163 LoadModule authz_user_module modules/mod_authz_user.so
 164 #LoadModule authz_owner_module modules/mod_authz_owner.so
 165 #LoadModule authz_groupfile_module modules/mod_authz_groupfile.so
 166 #LoadModule authz_dbm_module modules/mod_authz_dbm.so
 167 LoadModule authz_default_module modules/mod_authz_default.so
 168 #LoadModule ldap_module modules/mod_ldap.so
 169 #LoadModule authnz_ldap_module modules/mod_authnz_ldap.so
 170 LoadModule include_module modules/mod_include.so
 171 LoadModule log_config_module modules/mod_log_config.so
 172 LoadModule logio_module modules/mod_logio.so
 173 LoadModule env_module modules/mod_env.so
 174 #LoadModule ext_filter_module modules/mod_ext_filter.so
 175 LoadModule mime_magic_module modules/mod_mime_magic.so
 176 LoadModule expires_module modules/mod_expires.so
 177 LoadModule deflate_module modules/mod_deflate.so
 178 LoadModule headers_module modules/mod_headers.so
 179 #LoadModule usertrack_module modules/mod_usertrack.so
 180 LoadModule setenvif_module modules/mod_setenvif.so
 181 LoadModule mime_module modules/mod_mime.so
 182 #LoadModule dav_module modules/mod_dav.so
 183 LoadModule status_module modules/mod_status.so
 184 LoadModule autoindex_module modules/mod_autoindex.so
 185 LoadModule info_module modules/mod_info.so
 186 #LoadModule dav_fs_module modules/mod_dav_fs.so
 187 LoadModule vhost_alias_module modules/mod_vhost_alias.so
 188 LoadModule negotiation_module modules/mod_negotiation.so
 189 LoadModule dir_module modules/mod_dir.so
 190 #LoadModule actions_module modules/mod_actions.so
 191 #LoadModule speling_module modules/mod_speling.so
 192 #LoadModule userdir_module modules/mod_userdir.so
 193 LoadModule alias_module modules/mod_alias.so
 194 LoadModule rewrite_module modules/mod_rewrite.so
 195 #LoadModule proxy_module modules/mod_proxy.so
 196 #LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
 197 #LoadModule proxy_ftp_module modules/mod_proxy_ftp.so
 198 #LoadModule proxy_http_module modules/mod_proxy_http.so
 199 #LoadModule proxy_connect_module modules/mod_proxy_connect.so
 200 #LoadModule cache_module modules/mod_cache.so
 201 LoadModule suexec_module modules/mod_suexec.so
 202 #LoadModule disk_cache_module modules/mod_disk_cache.so
 203 #LoadModule file_cache_module modules/mod_file_cache.so
 204 #LoadModule mem_cache_module modules/mod_mem_cache.so
 205 LoadModule cgi_module modules/mod_cgi.so
 206 #LoadModule version_module modules/mod_version.so
 207 LoadModule ssl_module modules/mod_ssl.so
```


