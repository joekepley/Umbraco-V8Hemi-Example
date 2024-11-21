# V8 Hemi Umbraco Sample Package

This project is designed to serve as an example of how to build a project in Umbraco.
Umbraco's tutorials give you a great start in how to build a "Hello World" extension, but many of the
community-built packages use a more complex project structure, and it's not always clear _why_.

This package is intended to offer an opinionated example that is set up for long-term development across
multiple Umbraco versions. Hopefully it can serve as a bridge between "Hello World" packages and a
full-fledged maintainable project.

## What's In This Project

We're going to learn how to modify Umbraco in the same way decades of tinkerers have learned how to modify cars:
_**we're puttin' a Hemi in it!**_

![Screenshot of the Umbraco Admin with a custom UI featuring a V8 Hemi Engine](/assets/screenshot.png)

This is just a fun basic example that demonstrates the following extension points:

- Adding custom configuration
- Including custom services and controllers
- Adding a new OpenAPI-compatible endpoint, securing it, generating documentation and client code
- A custom dashboard view
- Custom Web Component using Lit
- How to structure a project for supporting multiple target versions

## Structuring an Umbraco Package Solution

Umbraco Packages are typically distributed via Nuget as Razor Class Libraries, and this can be done by simply creating
a single RCL project, loading it with all your code and assets, and publishing it to Nuget. However, there are
reasons that you may want to compartmentalize your code into multiple projects.

Umbraco projects after version 14 use the new [Project Bellissima][bel]-developed Umbraco Backoffice, which is a
JavaScript-based Single Page Application rather than the .Net MVC server-side rendered Razor template powered Backoffice
of previous versions (we'll refer to these as "Bellissima" and "Legacy"). This also means that many of the management
functions that were previously server-side .Net APIs are now exposed as REST APIs for consumption by the Bellissima
Backoffice (and your own code!). This greatly enhances the flexibility of the platform, but means that a package that
wants to support both Legacy and Bellissima Umbraco versions will need to implement different solutions for many
extension points.

For this reason, most of the more complex package solutions follow a project structure like this:

- **MyPackage**, a project that just references the right sub-projects based on the version being built. This is
  what gets added to Nuget
- **MyPackage.Common**, which contains the code that's common to all versions, like your internal services and class
  libraries. Many projects might also call this 'Core', or bundle it with the root project instead
- **MyPackage.Backend**, which contains the Bellissima-specific server side code. I've also seen this called things like
  MyPackage.V14 or MyPackage.Bellissima, but I'm recommending 'Backend' here. It's slightly confusing now, but once v13
  reaches end-of-life, this confusion will go away
- **MyPackage.Frontend**, which contains the Bellissima-specific client side code. Having this as its own project makes
  it easier to manage its own specific build pipeline. I've variously seen this called 'Client' or 'WebUI' as well
- **MyPackage.Legacy**, which contains the Legacy-specific code. I've also seen this referred to as a 'V13' project,
  makes some sense, even though it would also be how you might support earlier versions.
- **Version-Specific Test Packages**, which are installations of individual Umbraco versions with the package
  pre-referenced. These are instances of the Umbraco CMS intended to be used to allow rapid testing of the package on
  various Umbraco versions. These are for developers only and won't be shipped in the output package.

## Client-Side Code Philosophy

In the past, a lot of web-based CMS systems have had to choose an Official JavaScript Framework® for their admin UI.
Doing this well involves risk and a bit of fortune telling. Umbraco has attempted to avoid this through the use of
[Web Components][wc], a standards-compliant way of compartmentalizing code. In theory, this means that you can use _any_
framework, or even _no_ framework, to build your Umbraco Backoffice extensions. All the rest of the system expects is
Web Components, and provides a few extension points to use client-side services like authentication and API client code.

Practically speaking, there is still an "Umbraco Way" to extend the Backoffice, meaning that if you want to build
extensions that will be familiar to other Umbraco developers, you'll want to build your Web Components using the
[Lit Framework][lit], built by [Vite](vite), and written in [TypeScript](ts). You'll also want to reference the
[Umbraco UI][uui] components, and use [@hey-api/openapi-ts](hey) to auto-generate your API client code. None of this
is required, but it will align with a lot of the other example work available.

## Server-Side Code Philosophy

In Legacy Umbraco, the Backoffice uses a traditional .Net MVC Application: controllers that respond to requests,
manipulate Model objects, and generate Views using Razor templates, all of this happening on the server.

In Bellissima Umbraco, all of the actual UI happens on the client side. The server is instead providing REST API
endpoints for everything. This is done using the OpenAPI standard and [Swagger][swag] to decorate the .Net API Controllers,
which provides a lot of developer-friendly features.

If your goal is to reuse as much of your server-side code as you can across services, you should consider abstracting
your business logic out of controllers and in to Service classes that can be consumed by both Bellissima API Controllers
and Legacy controllers.

## Legacy Code Support

The consensus position among Umbraco developers at this point appears to be this: develop for Bellissima-era Umbraco,
and use various strategies to back-engineer support into Legacy Umbraco. Bellissima will be the way Umbraco is developed
for the foreseeable future. v13, the last Legacy version, will reach End-of-Life in December of 2026, so it makes sense
to look at Legacy as the 'temporary' target at this point.

The biggest accommodation that will need to be made is client-side code. Based on the project, you should ask yourself
whether it makes sense to try to build a separate traditional MVC UI alongside your Web Components UI, or if it makes
more sense to backport your server UI endpoints and Web Components into the Legacy UI. A couple of notes on approach,
based on the layer of the application:

- **Models and Services**: These should largely be cross-compatible across versions.
- **Controllers**: You could take one of two approaches here: Use APIControllers for Bellissima, and provide similar shim
  API Controllers on Legacy using UmbracoApiController classes, or build a separate "traditional" MVC approach using
  standard controllers.
- **Client UI**: Again, two approaches here: Either build a parallel UI just for Legacy, or you can
  [create an AngularJS wrapper][wcang] to reuse your Web Component in Angular JS. If you take this route, you'll need
  to account for the lack of supporting code in the Legacy Angular environment (primarily the lack of Contexts for
  things like authentication)

## Sources and Examples

This repository draws advice and inspiration from a lot of other example projects that you may find useful:

- **[Kevin Jump's TimeDashboard][kjtd]**: This project is Kevin's example from learning to work with Bellissima in the
  early stages. Kevin also has a really excellent series of [articles on building for Umbraco v14][kjguide].
  These are must-reads for new Umbraco package developers working with versions 14+
- **[Knowit's Bellissima Bootstrapper][knowit]**: Provides an auto-generated project with a layout similar to this one.
- **[Jesper Mayntzhusen's Extension Comparison][jmcompare]**: As part of CodeCabin, Jesper put together a short set of
  examples building the same extension in both Legacy and Bellissima. These are mostly built out in the most direct way
  possible and offer good comparisons for Legacy developers looking to see how the same thing might be built in Bellissima
- **[Richard Ockerby's Vanilla JS Example][rosimple]**: While most builds of Umbraco Backoffice extensions leverage
  complex frontend pipelines, Richard put together a _very_ basic example that shows that you can build a Bellissima
  backoffice extension using only standard JavaScript

Also, many thanks to these folks who offered advice and help, either directly or via educational content:

- Lee Kelleher
- Lotte Pitcher
- Richard Ockerby
- Jesper Mayntzhusen
- Matt Brailsford
- Sean Thorn
- Kevin Jump

## Technologies

For veteran Umbraco Devs, Bellissima makes use of a lot of new technologies that might be unfamiliar. Here are a number
of new technologies and some guides to help you get started:

- **[Web Components][wc]**: Web Components are a standardized approach to building self-contained and reusable custom
  functionality. Supported by all major browsers, they offer a standards-based approach to reusable frontend code that
  would have previously required frameworks like React, Vuejs, and etc
- **[Lit][lit]**: While you can build web components without using any framework, it's helpful to have some structure.
  Lit provides an extremely thin framework around web components functionality to reduce boilerplate and improve the
  developer experience.
- **[TypeScript][ts]**: TypeScript provides a more modern framework to write JavaScript in a way that provides a cleaner
  syntax, intellisense, and type safety, making writing JavaScript feel a little like writing a compiled language.
  Typescript compiles into JavaScript for deployment.
- **[Vite][vite]**: Vite provides a very fast modern build pipeline for developer tasks with front end code. You probably
  won't need the details on this at first, but understanding it is helpful for growing into more complicated build tasks.
- **[Swagger][swag]**: Umbraco v14+ uses the OpenAPI standard, which allows nice things to happen around APIs
  automatically. One of those is Swagger, which auto-generates documentation for API endpoints. Umbraco v14+ includes
  Swagger out of the box.
- **[Hey Api][hey]**: Another nice thing that OpenAPI allows for is automatic client code generation, meaning
  that your browser code can get type safety and intellisense for your API with minimal work. The @hey-api/openapi-ts
  package handles this code generation.

## Guides

If building packages is new to you, here are some resources to help you get started:

- **[Umbraco's Documentation][ug]**: Umbraco provides an excellent guide on creating your first extension
- **[Kevin Jump's Early Adopters Guide to v14][kj1]**: While we're no longer 'early', Kevin's series still provides a
  great starting point for someone who has written code for Umbraco in the past but is new to versions 14+.

[kjtd]: https://github.com/KevinJump/TimeDashboard
[kjguide]: https://dev.to/kevinjump/series
[knowit]: https://github.com/KXCPH/Knowit.Umbraco.Bellissima.Bootstrapper
[jmcompare]: https://github.com/jemayn/ExtensionComparisons
[rosimple]: https://github.com/Rockerby/Umbraco-14-Sample-Package
[wc]: https://www.webcomponents.org/introduction
[lit]: https://lit.dev/docs/getting-started/
[ts]: https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html
[vite]: https://vitejs.dev/guide/
[swag]: https://swagger.io/
[hey]: https://heyapi.vercel.app/
[uui]: https://github.com/umbraco/Umbraco.UI
[ug]: https://docs.umbraco.com/umbraco-cms/tutorials/creating-your-first-extension#extension-with-vite-typescript-and-lit
[kj1]: https://dev.to/kevinjump/early-adoptors-guide-to-umbraco-v14-package-structure-3i67
[bel]: https://docs.umbraco.com/umbraco-cms/customizing/project-bellissima
[wcang]: https://vibhas1892.medium.com/how-to-use-web-components-in-angular-applications-f82d430712eb

