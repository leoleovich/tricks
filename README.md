# There are some tips and tricks I want to save
* Rebuld pip to deb:
```
package='Flask-Profile-0.2'
py2dsc -m 'Oleg Obleukhov <leoleovich@something>' ./${package}.tar.gz && cd deb_dist/${package} && DEB_BUILD_OPTIONS=nocheck debuild -i -us -uc -b
```
