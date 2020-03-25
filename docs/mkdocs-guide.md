# Welcome to MkDocs

For full documentation visit [mkdocs.org](https://mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs help` - Print this help message.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

	mkdocs.yml sample
		site_name: MkDocs
		site_url: https://www.mkdocs.org
		site_description: Project documentation with Markdown.
		site_author: MkDocs Team
		
		repo_url: https://github.com/mkdocs/mkdocs/
		edit_uri: ""
		
		theme:
		    name: mkdocs
		    highlightjs: true
		    hljs_languages:
		        - yaml
		        - django
		
		nav:
		    - Home: index.md
		    - FTP:
		        - Setup: ftp/setup.md
		        - Config: ftp/config.md
		    - User Guide:
		        - Writing Your Docs: user-guide/writing-your-docs.md
		        - Styling Your Docs: user-guide/styling-your-docs.md
		        - Configuration: user-guide/configuration.md
		        - Deploying Your Docs: user-guide/deploying-your-docs.md
		        - Custom Themes: user-guide/custom-themes.md
		        - Plugins: user-guide/plugins.md
		    - About:
		        - Release Notes: about/release-notes.md
		        - Contributing: about/contributing.md
		        - License: about/license.md
		
		extra_css:
		    - css/extra.css
		
		markdown_extensions:
		    - toc:
		        permalink: 
		    - admonition
		    - def_list
		
		copyright: Copyright &copy; 2014 <a href="https://twitter.com/_tomchristie">Tom Christie</a>, Maintained by the <a href="/about/release-notes/#maintenance-team">MkDocs Team</a>.
		google_analytics: ['UA-27795084-5', 'mkdocs.org']
		
		plugins:
		    - search
		
