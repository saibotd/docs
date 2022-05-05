# Listing service

You can download a plugin showcasing the topic [here](https://docs.enterprise.shopware.com/exampleplugins/B2bAcl.zip).

## Table of contents
  
* [The Pattern](#the-pattern)
* [The Search Struct](#the-search-struct)
* [The Repository](#the-repository)
* [The Grid Helper](#the-grid-helper)

## The Pattern
A repeating pattern used throughout the B2B-Suite are listing services. The B2B-Suite ships without an ORM but still has use for semi automated basic listing and filtering capabilities. To reduce the necessary duplications, there are common implementations for this.

The diagram below shows the usually implemented objects with their outside dependencies.

![image](/.gitbook/assets/listing-service.svg)

## The Search Struct

The globally used `SearchStruct` is a data container moving the requested filter, sorting and pagination data from HTTP request to the repository/query.
```php
<?php

namespace Shopware\B2B\Common\Repository;

class SearchStruct
{
    /**
     * @var Shopware\B2B\Common\Filter\Filter[]
     */
    public $filters = [];

    /**
     * @var int
     */
    public $limit;

    /**
     * @var int
     */
    public $offset;

    /**
     * @var string
     */
    public $orderBy;

    /**
     * @var string
     */
    public $orderDirection = 'ASC';

    /**
     * @var string
     */
    public $searchTerm;
}
```

A more special `SearchStruct` is the `CompanyFilterStruct` (see [company](./company.md)).

## The Repository
The repository has to implement `Shopware\B2B\Common\Controller\GridRepository` and therefore have these three methods:

```php
<?php

class Repository implements Shopware\B2B\Common\Controller\GridRepository
{
    /**
     * @return string query alias for filter construction
     */
    public function getMainTableAlias(): string;

    /**
     * @return string[] database field names
     */
    public function getFullTextSearchFields(): array;

    /**
     * @return array
     */
    public function getAdditionalSearchResourceAndFields(): array;
}
```

But more important than that it has to handle the data encapsulated in `Shopware\B2B\Common\Repository\SearchStruct` and be able to provide a list of items and a total count of all accessible records.

```php
<?php

class Repository
{
    /**
     * @param OwnershipContext $context
     * @param ContactSearchStruct $searchStruct
     * @return array
     */
    public function fetchList(OwnershipContext $context, ContactSearchStruct $searchStruct): array
    {
        [...]
    }

    /**
     * @param OwnershipContext $context
     * @param ContactSearchStruct $contactSearchStruct
     * @return int
     */
    public function fetchTotalCount(OwnershipContext $context, ContactSearchStruct $contactSearchStruct): int
    {
        [...]
    }
}
```

Since this task is completely storage engine related there is **no further service abstraction** and every user of this functionality accesses the repository directly.

## The Grid Helper

The GridHelper binds the HTTP request data to the `SearchStruct` and provides the canonical build grid state array to be consumed by the frontend.

```php
<?php

class GridHelper
{

    /**
     * @param \Enlight_Controller_Request_Request $request
     * @param $struct
     */
    public function extractSearchDataInStoreFront(
        \Enlight_Controller_Request_Request $request,
        SearchStruct $struct
    );

    /**
     * @param \Enlight_Controller_Request_Request $request
     * @param SearchStruct $struct
     * @param array $data
     * @param int $maxPage
     * @param int $currentPage
     * @return array
     */
    public function getGridState(
        \Enlight_Controller_Request_Request $request,
        SearchStruct $struct,
        array $data,
        int $maxPage,
        int $currentPage
    ): array

}
```
