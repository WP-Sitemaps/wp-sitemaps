# Sitemaps feature project

## Definitions

In this document **sitemap(s)** referred to are computer-facing representations. A visual site representation, understandable by humans, can be extracted from the internal structure but is not going to be implemented in this project.

## Motivation

Sitemaps should be available for each website, they are an integral part of the internet. Sitemaps lead to better and easier indexation of websites by search engines. This results in higher visibility of the content. The specific format the sitemap is presented in should not be static. While XML has been the standard for the past, JSON is a much more future-proof format. 

### Background

"Sitemaps are an easy way for webmasters to inform search engines about pages on their sites that are available for crawling. In its simplest form, a Sitemap is an XML file that lists URLs for a site along with additional metadata about each URL (when it was last updated, how often it usually changes, and how important it is, relative to other URLs in the site) so that search engines can more intelligently crawl the site." - [sitemap definition](http://www.sitemaps.org/protocol.html) sitemaps.org

This definition was written in 2008 and describes the situation where websites used to have a "webmaster" who implements the technical requirements which are needed for a website.

The role of a "webmaster" has transferred partially to hosting providers and partially to content management systems. This means that the responsibility for the technical implementation of sitemaps is not clearly defined anymore.

When a website provides a sitemap, scrapers no longer have to read every page of the website in search of links to other pages. Providing the last modification date of the documents gives data mining services a better method to optimize the effort of analyzing the (changed) information on a website, thus also saving considerable amounts of bandwidth.

### Output format

Basic sitemaps consist of two different kind of documents.

* Sitemap-index
    * This document contains a list of all sitemaps this website offers.
    * Each sitemap has a last modified date.
* Sitemap
    * A document of links with the last modified time.

The sitemap definition describes priority and the indication of how often the referenced document will change, but these are optional and not being used in the extent that they were in the past.

Additionally Google supports [news](https://support.google.com/news/publisher/answer/74288?hl=en) and [video](https://developers.google.com/webmasters/videosearch/sitemaps) sitemaps. These sitemaps contain additional metadata about the items contained within. The specific metadata for the representations should be implemented by plugins which provide this specific need.

### Control over content

Every public-facing document needs to be in the sitemap. However there will always be exceptions. A landing page, temporary test page et cetera.

When the core system is responsible for generating sitemaps, there must be a way to exclude specific items or types of items from the sitemap. The system needs to be pluggable to allow developers to implement their own sitemap providers.

### Current implementations

Currently several major plugins have an XML sitemaps implementation: Jetpack, Yoast SEO, Google XML Sitemaps amongst others. It is a waste of time and effort that multiple teams are working on implementing this standard. Yoast has taken the initiative to lead this feature project, which will bring the functionality to WordPress core and is prepared to help maintain it thereafter.

## Insights and approach

Based on a vast expertise in the field of SEO present at Yoast and consultation with people at the upper ranks within Google Webmaster tools, we’ve come up with a performant and scalable approach towards building XML sitemaps.

### Architectural concept: Groups

As we’re potentially dealing with gigantic data volumes, the design needs to be done in a way that it keeps algorithmic complexity in check and allows easy partitioning of the data set.

To index the data set into a data model we can later use for building the sitemap, we first split it up into *groups*. These groups are our basic unit of work, to partition the data set up into more manageable chunks. They are also the first level at which the caching takes place.

They also come in handy when rendering the XML representation of our sitemap, to account for the technical limitations imposed by the XML specification (max. 50.000 items or less, max. 10MB size or less, per file).

The individual elements need to be distributed amongst groups. This is done by *labelling* these elements, effectively attaching the group ID to the element to be distributed. Indexation and distribution are two separate tasks. The indexation will determine the items that have not yet been distributed. The actual distribution is triggered on a specific request. Labelling the items and rendering the specific groups is a very cheap and quick process.

Items are labelled with the group they are distributed to. During initial indexation or after changing the maximum item count, all items need to be unlabelled and can be redistributed. The persistence mechanism for the labels needs to be chosen so that the unlabelling of entire groups can happen in one quick operation.

### Performance / Cache

The cache mechanism will be an implementation of the Russian Doll Caching principle.

* Overall hash: URL + date-time of last modification to the site
    * Group hash: group identifier + date-time of last modification to an item within that group
        * Item hash: item identifier + date-time of last modification to that item

Such a structure allows a quick invalidation of the caches and gradual discovery of content that needs to be reindexed. For installations without a persistent object cache, a regular clean-up task might need to be considered.

Index structures will be stored in temporary cache and generated when needed. Generated representations of the structure will be stored in static cache whenever available.

## Roadmap

The following subjects will be developed in separate proofs of concept:

* Indexing/re-indexing a whole site
    * What implications does this have
    * How do we keep this responsive and non-blocking
    * How do we make sure this does not take forever
* Serving sitemaps efficiently and fast
    * Can we have file-caching
    * Structure vs representation
* Group control
    * Adding/removing/updating items
    * Listing indexes efficiently
* Custom sitemap API
    * Allowing for developers to create their own sitemap and provide entries

When the complexity has been found and resolved, the concept will be done. The lessons learned and approach concluded will be used to build the initial version of the sitemaps feature project plugin.

## Optional: Multiple output formats

The structured data stored can easily be represented in multiple ways. We could, for instance, implement a JSON representation. This will provide the sitemaps in the JSON format which can be integrated more easily in modern systems.

The filesize of a JSON representation is significantly smaller compared to an XML representation. It is also easier, faster and thus cheaper to parse.

## Project activity

You can follow the project on Github, where the code and issues are.

Github: https://github.com/WP-Sitemaps/wp-sitemaps

For discussions and the weekly meeting, visit the WordPress Slack channel.

Channel: #core-sitemaps

Weekly meeting: Mondays at 15:00 UTC

## Contributors

The following people have helped out in the project and are much appreciated for it:

* [Jip Moors](https://profiles.wordpress.org/jipmoors)
* [Sergey Biryukov](https://profiles.wordpress.org/sergeybiryukov)
* [Alain Schlesser](https://profiles.wordpress.org/schlessera)

