### TOC
```bash
curl-scraper --pretty --xpath 'tag_content://div[@class="entry-content"]' https://bakapervert.wordpress.com/isekai-tensei-soudouki/ | tee pretty-toc.htm
sed -E -i.bak 's/(.*img .*) class=.*( src="[^"].*") .*( width=.*)/\1\2\3/g' pretty-toc.htm
sed -E 's/src="([^?].*)\?.*" /src="\1" /g' pretty-toc.htm > pretty-toc.html
```
### Chapters
```bash
sed -i.bak -E  's/".*(soudou[^\/].*)\/"/"chapters\/\1\.html"/' soudouki-toc.html
for x in $(grep illus source.txt); do y=$(echo "${x%%/}" | awk -F '/' '{print $NF}'); mkdir "$y"; for z in $(curl-scraper --xpath 'data-orig-file://img' $x); do w=$(echo "$z" | awk -F '/' '{print $NF}'); wget --no-clobber "$z" -O "$y/$w"; done; done
mkd volume0{1..9}
mkd volume{10..14}
for x in {1..9}; do mv chapters/*"-${x}-"* ./"volume0${x}"/illust; done
for x in {10..14}; do mv chapters/*"-${x}-"* ./"volume${x}"/illust; done
for x in $(grep -v illus source.txt); do y=$(echo "${x%%/}" | awk -F '/' '{print $NF".html"}'); curl-scraper --pretty --xpath 'tag_content://div[@class="entry-content"]' -o "./$y" "$x"; done
for x in {1..9}; do mv -v --no-clobber *"vol-${x}-"* "volume0${x}/"; done
for x in {10..14}; do mv -v --no-clobber *"vol-${x}-"* "volume${x}/"; done
find ./ -mindepth 2 -type f -name '*.html' -exec sed -i '/daddy/,/<\/html>/d' {} \;
for x in $(find -mindepth 2 -type f -name '*.html'); do echo -e '  </div>\n </body>\n</html>' >> "$x"; done
```
### Links
```bash
for x in {1..9}; do for y in $(grep "\-vol\-${x}\-" index.html | awk -F '"' '{print $2}'); do z="volume0${x}/$y"; sed -i "s|$y|$z|g" index.html; done; done
for x in {10..14}; do for y in $(grep "\-vol\-${x}\-" index.html | awk -F '"' '{print $2}'); do z="volume${x}/$y"; sed -i "s|$y|$z|g" index.html; done; done
for y in $(grep "soudouki\-chapter\-" index.html | awk -F '"' '{print $2}'); do z="volume15-web/$y"; sed -i "s|$y|$z|g" index.html; done
for x in $(find ./ -mindepth 2 -name '*.html'); do sed -i "s|https://bakapervert.wordpress.com/||g" $x; done
for x in $(find ./ -mindepth 2 -name '*.html'); do for y in $(sed -nE 's|.*"(soudouki-.*[^/].*)/".*|\1|p' "$x"); do sed -i "s|${y}/|${y}.html|g" "$x"; done; done
for x in $(find ./ -mindepth 2 -name '*.html'); do sed -i "s|isekai-tensei-soudouki/|../index.html|g" "$x"; done
```