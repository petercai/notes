# host an Eclipse update site on Github
_Forget the Github project releases feature, that won't work as a true update site (see notes at the end)._

To achieve what you want, you can create a Github repo, commit/push your p2 repository there and then serve it as an update site, using raw links. So for example, for the repository:

[https://github.com/some-user/some-repository/](https://github.com/some-user/some-repository/)

you can serve it as an update site using the link:

[https://github.com/some-user/some-repository/raw/master/](https://github.com/some-user/some-repository/raw/master/)

Notes: Yes, if you open the update site link in a browser, github will give you no file listings, but rather a 404. But that's fine. The Eclipse update site mechanism doesn't need the parent link to be valid. Instead Eclipse will directly look for `<update-site URL>/artifacts.jar` (or .xml) and from the information in artifacts.jar, it will itself discover the URLs of the other artifacts stored in the update site. AFAIK, at no point does the Eclipse update mechanism need the web server to do file listings of a directory.

Note2: if you use Github project releases, you can only attach a zipped p2 repository to it. That is not a proper update site because it is a static repository: there is no URL to which new releases can be uploaded to. Eclipse won't be able to automatically discover new updates, rather the user will need to download the zip for each new release he/she wants to update to. (Also with a proper update site, only the necessary artifacts for installation/update/query will be downloaded - a minor advantage)