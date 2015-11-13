---
layout: post
title: "Magento Secrets: Solr Disabled For Catalogs Including Tax"
date: 2015-11-13 23:00
comments: true
tags:
    - magento
---
*This article is the first in a series pointing out issues and design choices of [Magento](http://magento.com) so fundamental and unbelievable yet almost unknown in the Magento scene. Some of those I have checked with renowned Magento solution partners in the region and learned that they did not know about them either. That said, I am not taking credit of being first to point those out but am just sharing my experience which may be useful for others.*

In this case, the issue is about the use of [Apache Solr](http://lucene.apache.org/solr/) &ndash; its support a key feature distributed in [Magento Enterprise](http://magento.com/products/enterprise-edition/marketing-and-merchandising) which the company charges around **$18,000 per year and server** (as of 2015). Solr is an open-source search engine based on [Apache Lucene](http://lucene.apache.org/index.html) and allows hit highlighting, faceted search and caching. The open-source Community Edition of Magento relies on MySQL full text search to serve search requests. In Enterprise Edition, Magento offers a second, better solution to search with Solr. In fact, it can also be used to serve the product listing pages, layered navigation filters and search recommendations.

The problem is that for a certain tax setting is causing Solr to be disregarded for serving listing pages and layered navigation filters. This setting specifies that displayed prices include VAT which is legally required for many countries including the United Kingdom and Germany. The Solr adapter is implemented in a way that it falls back to the MySQL adapter when there is an unexpected issue such as connection failure. If Solr is working partially &ndash; in this case for search or stores excluding tax &ndash; one might be left under the impression that all is going through Solr. Below is an excerpt from the code responsible for this:

~~~php
# Enterprise_Search_Helper_Data
/**
 * Check if search engine can be used for catalog navigation
 *
 * @param   bool $isCatalog - define if checking availability for catalog navigation or search result navigation
 * @return  bool
 */
public function getIsEngineAvailableForNavigation($isCatalog = true)
{
    if (is_null($this->_isEngineAvailableForNavigation)) {
        $this->_isEngineAvailableForNavigation = false;
        if ($this->isActiveEngine()) {
            if ($isCatalog) {
                if ($this->getSearchConfigData('solr_server_use_in_catalog_navigation')
                    && !$this->getTaxInfluence()
                ) {
                    $this->_isEngineAvailableForNavigation = true;
                }
            } else {
                $this->_isEngineAvailableForNavigation = true;
            }
        }
    }

    return $this->_isEngineAvailableForNavigation;
}
~~~

This constraint is nowhere documented nor mentioned in Sales contacts which were from the UK and aware of our strong intention to use Solr throughout the site. I could not believe this and contacted Magento Support in July 2014. Their response (see below) made me even more speechless:

    Hello,

    Thank you for contacting Magento Technical Support.

    My name is _____ and I will be happy to assist you.

    I am sorry, but I have to let you know that, yes, the behavior is as you described.
    It's caused by specifics of fulltext search engine usage, that is restricted in
    this case and can't guarantee the correct price filtering with dynamically
    modified price. Please note that it affects only catalog pages. Search will
    still work using solr, but price filter will be disabled in that case.

    Please let me know if you require further assistance.

    Best Wishes,
    _________
    Tech Support Analyst

In one project, I overrode the logic above so that it always relies on Solr regardless of the display type of taxes and no one actually noticed issues. We suggested that performance is more significant than some filter.
