# Frontend Guidelines
A one-page questionnaire to help your team establish effective frontend guidelines, so that you can write consistent & cohesive code together.

## HTML
### HTML Principles
- **TODO: What are some general principles your team should follow when writing HTML?** *(for example, authoring semantic HTML5 markup, accessibility, etc. See [these](http://www.yellowshoe.com.au/standards/#html) [resources](http://codeguide.co/#html) for [inspiration](http://manuals.gravitydept.com/code/html))*


### HTML Tools
- The Globe system uses [Freemarker Templates](http://freemarker.incubator.apache.org) for rendering its markup.
- Globe's core Freemarker macros are an important part of the system, and will automatically add certain classes to the HTML markup:
	- **`<@location>`** -- By default, the `<@location>` macro renders as a `<div>` element with two classes: 
 		- `loc` (for targeting all locations)
   		- `loc-{name}`, where `{name}` is the value of the macro's `name` attribute
 	- **`<@component>`** -- the standard wrapper component renders as a `<div>` element with several attributes based on the component configuration XML:
  		- all components recieve the `comp` class for narrowing selectors
  		- the component inheritence chain will all be rendered as class names. For example, given the following components
    	
			```
			<component name="foo" location="foo.ftl"/>
			<component name="bar" ref="foo"/>
			<component name="shaz" ref="bar"/>
			```
			and a foo.ftl that looks like this:
			```
			<@component>I am here</@component>
			```
			component `shaz` will render as
			```
			<div class="comp foo bar shaz">I am here</div>
			```
		- the component's id will be generated based on its name or id, plus an ordinal number. If the `bar` component above was included twice on a page, the HTML would render as
  
			```
			<div id="bar-1" class="comp foo bar">I am here</div>
			<div id="bar-2" class="comp foo bar">I am here</div>
			```
	- **`<html>`** -- the `<html>` tag will be rendered with a number of classes that will be removed by Modernizr to indicate support:
 		- no-js
   		- _**TODO: others**_

### HTML Style
- Globe recommends 4-space indentation for your HTML.
- Unless it is specifically intended for debugging, you should use Freemarker comments `<#-- --#>` instead of HTML comments. 
- Use whitespace lines sparingly, to separate blocks of related code or behavior.

---------------

## CSS 

### CSS Methodology
- We are taking our cues from the [SMACSS](https://smacss.com/) system, preferring the long-form `layout-` class prefix for layout components.
- Mantle-level components will often, additionally, be prefixed with `mntl-`. For example a "row" layout component that lived in the Mantle project would have the name (and class) `mntl-layout-row`.

### CSS Principles
- To take best advantage of Globe's modular nature, rules for a component's default behavior should be namespaced using the component's name/id. This will prevent accidental rule spill while still allowing the behavior to be overridden if necessary based on context.

	Selectors for component default behavior should be as short and specific as possible. Use direct child selectors where possible rather than descendant selectors.
	
	For example, a component called `byline` might look like this:
	```
	<!-- byline.ftl -->
	<@component> <!-- this would be given the "comp" and "byline" classes -->
		<img class="byline-image" src="..."/>
		<div class="byline-name">...</div>
		<div class="byline-site">...</div>
	</@component>
	```
	The corresponding SCSS should be structured in `byline.scss` as
	```
	.byline {
		border-left: 1px solid black;
		> .byline-image {
			float: left;
		}
		> .byline-name {
			font-weight: bold;
		}
		> .byline-site {
			font-size: .8em;
		}
	}
	```
- If there are *general* extension/toggle classes for the component, they can be expressed in this file:
	```
	.byline {
		[...]
		&.is-minified {
			border-left-width: 0px;
			> *:not(.byline-name) {
				display: none;
			}
		}
	}
	```

- However, if the component is extended via configuration, additional new behavior should live in a new SCSS file and be namespaced based on the extension class name, e.g.:
	```
	<component name="byline-supergood" ref="byline">
		<!-- will get both byline and byline-supergood as class names -->
		<stylesheet>byline-supergood.css</stylesheet>
	</component> 
	```
	where byline-supergood.scss:
	```
	.byline-supergood {
		border: 3px dashed red; /* the border is what makes it special! */
	}
	```
	_**OPEN QUESTION: should namespace rules include the full extension class chain -- e.g. `.byline.byline-supergood`, `.foo.bar.shaz` -- or just the extension class?**_

- When you want to modify or extend a component's default behavior when included in a specific component, the modifying rule should live in the *parent* component's CSS file, under the top-level (namespacing) rule:
	```  
  	.content-header {
		> .byline {
			width: 50%; /* or a @column-span mixin */
		}
	}
	```	
	Only where necessry (e.g., because the component is included in multiple different locations), should you reference the location in the rule.
- As a general rule, components should be designed to be fluid by default, and therefore unopinionated regarding width. Width is more likely to be set in layouts, or (as in the example above) for specific inclusions ("instances") of the component.
- **What are some general principles your team should follow when writing CSS?** *(For example, modularity, avoiding long selector strings, etc. See [these](http://cssguidelin.es/) [resources](http://www.yellowshoe.com.au/standards/#css) [for](http://manuals.gravitydept.com/code/css) [inspiration](http://codeguide.co/#css))*

### CSS Tools
- Globe uses [Sass](http://sass-lang.com/) as a preprocessor, using SCSS syntax.
- **What are the guidelines for using that preprocessor** *(check out [Sass Guidelines](http://sass-guidelin.es/) for inspiration)*?
- We are using a custom CSS reset, generated via [http://html5reset.org/](http://html5reset.org/), that combines elements from [Eric Meyer's reset](http://meyerweb.com/eric/tools/css/reset/), [HTML5 Doctor's reset](http://html5doctor.com/html-5-reset-stylesheet/), and HTML5 Boilerplate. 
- We are not using a CSS post-processor at the moment. *Globe will be moving to [Autoprefixer](https://github.com/postcss/autoprefixer) in the near future*.
- Globe automatically determines what CSS files are needed for a given page render based on the component hierarchy.  By default that CSS is rendered minified and concatenated. *Adding "[critical CSS](https://www.smashingmagazine.com/2015/08/understanding-critical-css/)" based on position within that hierarchy may be a future addition, but it is not supported today.*
- Element Media Queries are allowed.  The preferred library is [Marc Schmit's CSS Element Queries](https://github.com/marcj/css-element-queries). *This will be supported by default via Mantle soon, but for now it needs to be included manually.*

### CSS Frameworks
- We are using [Susy](http://susy.oddbird.net/) for our grid layout mixins. Example usage may be found in the mantle-ref project at http://localhost:8080/grid

### CSS Style
- Indentation should be 4 spaces, not tabs
- Rulesets should, whenever possible, be organized to match the DOM structure  they reference.
- Use whitespace as necessary to logically separate rulesets. Properties themselves should generally not be separated. 
- Properties should be organized alphabetically within a rule. 

	_**OPEN QUESTION: or should they? Here is an alternate [grouping](https://smacss.com/book/formatting#grouping) methodology**_
- Most comments should not appear in the rendered CSS. Following the [SCSS comment style](http://sass-lang.com/documentation/file.SCSS_FOR_SASS_USERS.html#comments), use `//` for most comment blocks, and `/* */` if you want the comments to appear. 
- _**OPEN QUESTION: CSS documentation comments?**_

---------------

## JavaScript

### JavaScript Principles
- Follow the principle of progressive enhancement. As much as possibly, JS should *augment* functionality, not be required in order to provide it.
- **TODO: What are some general principles your team should follow when writing JavaScript?** *(See [these](https://github.com/airbnb/javascript) [resources](https://github.com/rwaldron/idiomatic.js) for [inspiration](https://github.com/styleguide/javascript))*


### JavaScript tools
- Globe uses [jQuery 1.11.0](http://jquery.com/), although this may change if we end A-level support for IE8.
- Globe uses [Modernizr](https://modernizr.com/) with auto-feature-detect integration.  Current implemented polyfills/features include:
	- _**TODO**_
- There are a number of additional plugins that are being managed by Bower. Please check [bower.json](https://stash.ops.about.com/projects/FRON/repos/mantle/browse/mantle-resources/src/main/resources/bower/bower.json?at=refs%2Fheads%2Ffeature%2Fvertical-epic) for specifics.

### JavaScript Style 
*(See [these](https://github.com/airbnb/javascript) [resources](https://github.com/rwaldron/idiomatic.js) for [inspiration](https://github.com/styleguide/javascript))*
- Indentation should be 4 spaces, no tabs.
- **TODO: What does JS commenting look like?** 
- **TODO: [What patterns are you following](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)?**

---------------

## Tooling
- We use [Grunt](http://gruntjs.com/) as a front-end build tool.  It handles CSS compilation, CSS+JS uglification, and sprite generation.
- We use [Bower](http://bower.io/) to manage 3rd-party SCSS/JS/CSS packages.
- JS is automatically Linted.
- For server-side tools and installation, please see the [Mantle feature/vertical-epic README](https://stash.ops.about.com/projects/FRON/repos/mantle/browse/README.md?at=refs%2Fheads%2Ffeature%2Fvertical-epic) 

---------------

## Version control
- We are using [Git](https://git-scm.com/) for version control, via a Bitbucket/Stash server hosted at [https://stash.ops.about.com](https://stash.ops.about.com)
- `master` is our deployment branch; snapshots are taken off this branch.  `develop` is deployed to the primary QA server for testing. We generally follow a [feature-branch](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) workflow.  Branch names should follow these patterns:
	- **feature**: `feature/{JIRA-ID}_{descriptive-name}`
	- **bugfix**: `bugfix/{JIRA-ID}_{descriptive-name}`
	- **hotfix**: `hotfix/{1.2.3}`
 	- **spike** or **POC**: `spike/{descriptive-name}`
- **Devs** are responsible for creating feature and bugfix branches, and creating pull requests. **QA** is responsible for merging approved requests.  This allows them to manage the flow of unvetted work and keeps `develop` closer to a state of deployment readiness. Unless you have specific approval from QA, nothing should be directly pushed to `develop` or `master`.
- Issues are tracked in [JIRA](https://jira.corp.about-inc.com/).

-----------

## Support and Optimization
It's important to recognize the difference between ["support" and "optimization"](http://bradfrost.com/blog/mobile/support-vs-optimization/). You should do your best to support as many environments as possible while simultaneously optimizing for the environments that make the most sense for your business and users. 

- We optimize for the following browsers using a [graded scale](https://github.com/yui/yui3/wiki/Graded-Browser-Support):
	- A-grade
		- **IE** (10, 11, Edge | Windows)
		- **Firefox** (current - 3 | All)
 		- **Chrome** (current - 3 | All)
  		- **Safari** (current | MacOS, iOS)
    - B-Grade
    	- **Opera**
     	- **Android Native**
    - C-Grade
    	- **IE** (8, 9)
     	- **Opera Mini**
     	- **Safari** (all other OS)
     	- **Blackberry**
    
- We optimize for Windows 7+, MacOS, and Ubuntu desktop machines, as well as iOS 4+ and Android 4.3+ mobile/tablet devices.

-----------

## Documentation
- Globe includes built-in pattern library functionality at the Mantle layer. Your workflow for creating and documenting a new component would probably work something like this:
	- **Build a "grey-box" version of your component.**  Add defaults for all dynamic properties/models that the component requires. *TODO: If a property is optional, stub it without a value; e.g., `<property name="optional"/>`*. *TODO: If the property is expected to have a finite set of values (i.e., being used in a switch statement, add `type="enum[x,y,z]"`*

		We recommend using the folling placeholders at this point: [placehold.it](https://placehold.it) for images, Lorem Ipsum for text, and the `json` model for complex objects).  Layout and styling are optional here.
 	- **Add documentation to your `<component>` tag.**
     	- `<category>` -- this is the trigger that will cause the component's inclusion in the PL. Supports a nesting tree structure by way of `/` characters.
  		- `<displayName>` -- pretty name
    	- `<description>` -- largely for product
     	- `<documentation>` -- largely for devs
   - **Access your component at {server}/pattern-library/{category}.**  Spaces in the `category` will be converted to dashes for the URL. Additional documentation will be autogenerated regarding stylesheets, component xml, required properties, etc.
   - **Add styling to the component and iterate in the PL!**
- Specific startup documentation should live in your project's README.md file
- **Where does your documentation live**? What are the links to the documentation?
- **Who's responsible for maintaining and governing the documentation**?
- **What happens when the guidelines are updated**?

-----------

*Feel free to modify or extend (such as adding specific sections for performance, accessibility, etc) this document for your own organization's needs. For questions, comments, additions, and corrections, please open an issue on Github and/or reach out to [@brad_frost](https://twitter.com/brad_frost) on Twitter.*
