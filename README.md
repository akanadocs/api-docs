# Updating the API Docs	
To optimize Github Pages build times (we were getting timeouts) we have moved the CM API Docs (/cm/api) to a separate repository and have replaced them with pre-built versions.  There are a few steps to go through when we need to update the API docs on the test and production servers.  All CM API Documentation updates should be made in the [API Docs repository](https://github.com/akanadocs/api-docs).

## Updating docs-test
1. Make sure your docs-test and api-docs repositories are up-to-date:

	```
	cd ~/projects/akanadocs/docs-test
	git pull
	cd ~/projects/akanadocs/api-docs
	git pull
	```
	
1. Copy the newly refreshed docs-test into a new temporary location (I keep my repositories in /projects/akanadocs in my home directory):

	```
	cd ..
	rm -rf temp
	cp -R docs-test temp
	```

1. Remove the pre-built API docs from this temp folder, and replace them with the docs from your API Docs repo, then build the temp repo:

	```
	rm -rf temp/cm/api temp/cm/api_oauth
	cp -R api-docs/cm/api/ temp/cm/api/
	cp -R api-docs/cm/api_oauth/ temp/cm/api_oauth/
	cd temp
	bundle exec jekyll build
	```

	Once the build completes you have a couple more steps:

1.	Replace the ```cm/api``` and ```cm/api_oauth``` folders om **docs-test** with the ones from the built _site in **temp**:

	```
	cd ..
	rm -rf docs-test/cm/api/ docs-test/cm/api_oauth/
	cp -R temp/_site/cm/api/ docs-test/cm/api/
	cp -R temp/_site/cm/api_oauth/ docs-test/cm/api_oauth/
	```
	
1. 	Override the pre-built API docs index files with the original ones from the api-docs repo
	
	```
	cp api-docs/cm/api/index.htm docs-test/cm/api/
	cp api-docs/cm/api_oauth/index.htm docs-test/cm/api_oauth/
	```

1.	Replace the content of the ```unsorted_pages_prod_cat``` variable in ```_includes/unsorted-pages-apioverview``` in your **docs-test** repo, with the contents of the same variable taken from ```_site/cm/api/Ref_API_Reference.htm```, or any of the Ref docs in your **temp** repo - now in docs-test too.

This is an easy step, but does require care.  Make sure you copy the entire unsorted_pages_prod_cat variable declaration (from var through the semicolon) ***FROM*** a file in temp ***TO*** the ```_includes/unsorted-pages-apioverview``` in ***docs-test***.

	> NOTE: this step is essential for maintaining the navigation structure of the API Docs.

1.	Push the updated API Docs up to docs-test:

	> NOTE: Make sure you change the placeholder text in the commit!!

	```
	cd docs-test
	git add -A .
	git commit -m "<JIRA # - Update reason>"
	git push
	```
	
And you're done.

## Updating the production site
This process has changed, you need to follow the exact same process as above, but for akanadocs.github.io instead of docs-test.  Run the exact same command sequence and steps.

1.	Do all the same steps as for docs-test.  Here are the Linux commands:

	```
	cd ~/projects/akanadocs/akanadocs.github.io
	git pull
	cd ~/projects/akanadocs/api-docs
	git pull
	cd ..
	rm -rf temp
	cp -R akanadocs.github.io temp
	rm -rf temp/cm/api temp/cm/api_oauth
	cp -R api-docs/cm/api/ temp/cm/api/
	cp -R api-docs/cm/api_oauth/ temp/cm/api_oauth/
	cd temp
	bundle exec jekyll build
	cd ~/projects/akanadocs
	rm -rf akanadocs.github.io/cm/api/ akanadocs.github.io/cm/api_oauth/
	cp -R temp/_site/cm/api/ akanadocs.github.io/cm/api/
	cp -R temp/_site/cm/api_oauth/ akanadocs.github.io/cm/api_oauth/
	cp api-docs/cm/api/index.htm akanadocs.github.io/cm/api/
	cp api-docs/cm/api_oauth/index.htm akanadocs.github.io/cm/api_oauth/
	```

1. Replace the content of the ```unsorted_pages_prod_cat``` variable in ```_includes/unsorted-pages-apioverview``` in your **akanadocs.github.io** repo, with the contents of the same variable taken from ```_site/cm/api/Ref_API_Reference.htm```, or any of the Ref docs in your **temp** repo - now in akanadocs.github.io too.

	> NOTE: this step is essential for maintaining the navigation structure of the API Docs.

1.	Push the updated API Docs up to akanadocs.github.io:

	> NOTE: Make sure you change the placeholder text in the commit!!

	```
	cd akanadocs.github.io
	git add -A .
	git commit -m "<JIRA # - Update reason>"
	git push
	```
