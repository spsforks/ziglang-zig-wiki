How to set up a server to run the ziglang.org website.

```
apt install python3-certbot-nginx
chown -R www-data: /var/www/html/
chmod -R g+w /var/www/html/
usermod -a -G www-data ci
```

Edit `/etc/nginx/sites-available/ziglang.org` with this configuration:

```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.html;

        server_name ziglang.org;

        gzip on;
        gzip_types
                application/javascript
                application/json
                application/x-tar
                application/xml
                application/xml+rss
                text/css
                text/html
                text/javascript
                text/plain
                text/xml
        ;
        gunzip on;
        gzip_static always;


        location / {
        }

        types {
            text/html                             html htm shtml;
            text/css                              css;
            text/xml                              xml;
            image/gif                             gif;
            image/jpeg                            jpeg jpg;
            application/javascript                js;
            application/atom+xml                  atom;
            application/rss+xml                   rss;

            text/mathml                           mml;
            text/plain                            txt;
            text/vnd.sun.j2me.app-descriptor      jad;
            text/vnd.wap.wml                      wml;
            text/x-component                      htc;

            image/png                             png;
            image/tiff                            tif tiff;
            image/vnd.wap.wbmp                    wbmp;
            image/x-icon                          ico;
            image/x-jng                           jng;
            image/x-ms-bmp                        bmp;
            image/svg+xml                         svg svgz;
            image/webp                            webp;

            application/font-woff                 woff;
            application/java-archive              jar war ear;
            application/json                      json;
            application/mac-binhex40              hqx;
            application/msword                    doc;
            application/pdf                       pdf;
            application/postscript                ps eps ai;
            application/rtf                       rtf;
            application/vnd.apple.mpegurl         m3u8;
            application/vnd.ms-excel              xls;
            application/vnd.ms-fontobject         eot;
            application/vnd.ms-powerpoint         ppt;
            application/vnd.wap.wmlc              wmlc;
            application/vnd.google-earth.kml+xml  kml;
            application/vnd.google-earth.kmz      kmz;
            application/wasm                      wasm;
            application/x-7z-compressed           7z;
            application/x-cocoa                   cco;
            application/x-java-archive-diff       jardiff;
            application/x-java-jnlp-file          jnlp;
            application/x-makeself                run;
            application/x-perl                    pl pm;
            application/x-pilot                   prc pdb;
            application/x-rar-compressed          rar;
            application/x-redhat-package-manager  rpm;
            application/x-sea                     sea;
            application/x-shockwave-flash         swf;
            application/x-stuffit                 sit;
            application/x-tcl                     tcl tk;
            application/x-x509-ca-cert            der pem crt;
            application/x-xpinstall               xpi;
            application/xhtml+xml                 xhtml;
            application/xspf+xml                  xspf;
            application/zip                       zip;

            application/octet-stream              bin exe dll;
            application/octet-stream              deb;
            application/octet-stream              dmg;
            application/octet-stream              iso img;
            application/octet-stream              msi msp msm;

            application/vnd.openxmlformats-officedocument.wordprocessingml.document    docx;
            application/vnd.openxmlformats-officedocument.spreadsheetml.sheet          xlsx;
            application/vnd.openxmlformats-officedocument.presentationml.presentation  pptx;

            audio/midi                            mid midi kar;
            audio/mpeg                            mp3;
            audio/ogg                             ogg;
            audio/x-m4a                           m4a;
            audio/x-realaudio                     ra;

            video/3gpp                            3gpp 3gp;
            video/mp2t                            ts;
            video/mp4                             mp4;
            video/mpeg                            mpeg mpg;
            video/quicktime                       mov;
            video/webm                            webm;
            video/x-flv                           flv;
            video/x-m4v                           m4v;
            video/x-mng                           mng;
            video/x-ms-asf                        asx asf;
            video/x-ms-wmv                        wmv;
            video/x-msvideo                       avi;
        }
}
```

This makes it so that any files named `foo.gz` are also fetchable as `foo` with
`Accept-Encoding: gzip`, which is used by the tarball script for `sources.tar`
and `main.wasm`.

```
cd /etc/nginx/sites-enabled
ln -s /etc/nginx/sites-available/ziglang.org
mv ziglang.org default
systemctl restart nginx
su ci
rm /var/www/html/index.nginx-debian.html
certbot --nginx -d example.com
```


