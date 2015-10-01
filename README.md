# Updating the API Docs	
To optimize Github Pages build times (we were getting timeouts) we have moved the CM API Docs (/cm/api) to a separate repository and have replaced them with pre-built versions.  There are a few steps to go through when we need to update the API docs on the test and production servers.  All CM API Documentation updates should be made in the [API Docs repository](https://github.com/akanadocs/api-docs).

## Updating docs-test
1. Make sure your docs-test and api-docs repositories are up-to-date with ```git pull```
1. Clone docs-test into a new temporary repository (I keep my repositories in /projects/akanadocs in my home directory):

	```
	rm -rf temp
	cd ~/projects/akanadocs
	git clone https://github.com/akanadocs/docs-test temp
```

1. Remove the pre-built API docs from this temp repo, and replace them with the docs from your API Docs repo, then build the temp repo:

	```
	rm -rf temp/cm/api
	cp -R api-docs/cm/api/ temp/cm/api/
	cd temp
	bundle exec jekyll build
	```

	Once the build completes you have a couple more steps:

1.	Save the ```/cm/api/index.htm``` file from the **docs-test** repo somewhere safe:

	```
	cd ~/projects/akanadocs
	cp docs-test/cm/api/index.htm .
	```

1.	Replace the ```cm/api`` folder om **docs-test** with the one from the built _site in **temp**:

	```
	rm -rf docs-test/cm/api/
	cp -R temp/_site/cm/api/ docs-test/cm/api/
	```
	
1.	Put the saved ```index.htm``` file back:

	```
	cp index.htm docs-test/cm/api/
	```

1. Replace the content of the ```unsorted_pages_prod_cat``` variable in ```_includes/unsorted-pages-apioverview``` in your **docs-test** repo, with the contents of the same variable taken from ```_site/cm/api/Ref_API_Reference.htm```, or any of the Ref docs in your **temp** repo - now in docs-test too.

	> NOTE: this step is essential for maintaining the navigation structure of the API Docs.

1.	Push the updated API Docs up to docs-test:

	```
	cd docs-test
	git add -A .
	git commit -m "<JIRA # - Update reason>"
	git push
	```
	
And you're done.

## Updating the production site
This is much easier, all you need to do is replace the /cm/api folder in the product site with the /cm/api folder from docs-test:

```
cd ~/projects/akanadocs
cd docs-test
git pull
cd ../akanadocs.github.io
git pull
rm -rf cm/api
cp -R ../docs-test/cm/api/ cm/api
git add -A .
git commit -m "<JIRA # - Update reason>"
git push
```

And that's it.

## Updating the search index
The script to update the search index runs at 8am GMT.  It is located on the ```blog.akana.com``` server at: ```/home/igoldsmith/build_akana_index.sh```.  The script is:

```
#!/bin/bash

export LANG=en_US.UTF-8
source /etc/profile
source /home/igoldsmith/.profile
cd /home/igoldsmith/akanadocs/akanadocs.github.io
/usr/bin/git pull
cd /home/igoldsmith/akanadocs/api-docs
/usr/bin/git pull
rm -rf /home/igoldsmith/akanadocs/build
cp -R /home/igoldsmith/akanadocs/akanadocs.github.io /home/igoldsmith/akanadocs/build
rm -rf /home/igoldsmith/akanadocs/build/cm/api
cp -R /home/igoldsmith/akanadocs/api-docs/cm/api/ /home/igoldsmith/akanadocs/build/cm/api/
cd /home/igoldsmith/akanadocs/build
/usr/local/rvm/gems/ruby-2.0.0-p353/bin/bundle exec jekyll build
cd /home/igoldsmith/akanadocs/akanadocs.github.io
cp /home/igoldsmith/akanadocs/build/search.json .
/usr/bin/git add search.json
/usr/bin/git commit -a -m "Update search index - $(date)"
/usr/bin/git push
```

The script shows how the build construct works to combine the primary and the non-pre-built docs repo.

Note that this script uses the Git credentials for the logged in user (the CRON job runs the command as igoldsmith).



