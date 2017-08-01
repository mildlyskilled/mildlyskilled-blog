---
title: "Full Text Indexing with Zend_Search_Lucene"
date: 2010-07-30T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["Zend", "PHP"]
---

To provide a robust search facility for content you have stored on a MySQL database, you may be thinking of using MySQLs builting full text indexing facility. This comes with the MyISAM storage engine and can be slightly limiting especially if you would like to use foreign keys and such. So queries such has the following:


     "SELECT *, MATCH (page_content, page_title) AGAINST
    (:keywords IN BOOLEAN MODE) AS relevance
    FROM pages WHERE MATCH (page_content, page_title) AGAINST
    (:keywords2 IN BOOLEAN MODE) ORDER BY relevance DESC";


may not work for you although this may be how you want to go.

The best way I have found is to leave the indexing to another service in general. Enter Zend_Search_Lucene, i believe this is a PHP port of the Apache Lucene project written in Java....

So basically we start by creating our index and document, personally i like to remove my personal classes and all that from the Zend Application as often as I can that way it's mostly untainted and I can then use the same Zend Application with multiple custom code bases, enough on that already. However you chose to go you need a version of the following:


    class My_SearchService {

        protected
        $indexPath,
        $pageService,
        $pageIndexPath,
        $newsIndexPath,
        $document,
        $pageIndex;

        public function setIndexPath($indexPath) {
            $this->indexPath = $indexPath;
        }

        public function __construct($indexPath = NULL) {
            if (is_null($indexPath)) {
                $indexPath = APPLICATION_PATH . '/indexes/';
            }
            $this->setIndexPath($indexPath);
            $this->pageIndexPath = $this->indexPath . 'pageindex';
            $this->newsindexPath = $this->indexPath . 'newsindex';
            $this->pageService = new My_PageService();
        }

        public function createPageIndex() {
            $this->pageIndex = Zend_Search_Lucene::create($this->pageIndexPath);
    // this is a simple Zend_Db_Table object returning data from my database
            $pages = $this->pageService->getAllPages()->toArray();

            foreach ($pages as $page) {
                $this->pageIndex
    ->addDocument(new My_Controller_Plugin_PageIndexer($page));
            }

            // commit index
            $this->pageIndex->commit();
        }

    }


This is assuming you have a folder structure that looks a bit like

/applications
--/everything else here
/library
--/My
--/PageService.php
-----/Controller
----------/Plugin
--------------/PageIndexer.php

Our PageIndexer is just out extension of Zend_Search_Lucene_Document this is what we will be querying for all our page search needs.


    /**
     * Description of PageIndexer
     *
     * @author kaning
     */
    class STEMNET_Controller_Plugin_PageIndexer extends Zend_Search_Lucene_Document {

        /**
         * Constructor. Creates our indexable document and adds all
         * necessary fields to it using the passed in document
         */
        public function __construct($document) {
           $this->addField(Zend_Search_Lucene_Field::Keyword('page_id', $document['page_id']));
            $this->addField(Zend_Search_Lucene_Field::UnIndexed('name', $document['page_name']));
            $this->addField(Zend_Search_Lucene_Field::UnIndexed('created', $document['publishdate']));
            $this->addField(Zend_Search_Lucene_Field::UnIndexed('caption', $document['page_caption']));
            $this->addField(Zend_Search_Lucene_Field::Text('title', $document['page_title']));
            $this->addField(Zend_Search_Lucene_Field::UnStored('content', $document['page_content']));
        }

    }


Not much introduction here besides the fact that I am adding the fields I want in my index bear in mind that the field data I am adding simply corresponds to what the Zend_Db_table select query returned for me.

Bear in mind that you really need to create the index only once. You will be updating it in subsequent times.

I formed the basis of this post from this [tutorial.](http://www.phpriot.com/articles/zend-search-lucene) Look it up for more information


class STEMNET_SearchService {

protected
$indexPath,
$pageService,
$pageIndexPath,
$newsIndexPath,
$document,
$pageIndex;

public function setIndexPath($indexPath) {
$this->indexPath = $indexPath;
}

public function __construct($indexPath = NULL) {
if (is_null($indexPath)) {
$indexPath = APPLICATION_PATH . '/indexes/';
}
$this->setIndexPath($indexPath);
$this->pageIndexPath = $this->indexPath . 'pageindex';
$this->newsindexPath = $this->indexPath . 'newsindex';
$this->pageService = new STEMNET_PageService();
}

public function createPageIndex() {
$this->pageIndex = Zend_Search_Lucene::create($this->pageIndexPath);
$pages = $this->pageService->getAllPages();

foreach ($pages as $page) {
$this->pageIndex->addDocument(new STEMNET_Controller_Plugin_PageIndexer($page));
}

// commit index
$this->pageIndex->commit();
}

}
