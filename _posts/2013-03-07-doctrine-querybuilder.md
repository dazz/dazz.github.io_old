---
layout: post
category : php
tagline: "Doctrine QueryBuilder"
tags : [doctrine, querybuilder, architecture, tutorial]
---
{% include JB/setup %}

# Doctrine QueryBuilder

Whenever I started a new project and used the Doctrine ORM in the past, I started to write methods in the Entity repository classes to get all the necessary data for the logic and the views.

At first it seemed simple when I wrote a `findAll` function. But as the project grew, a ton of parameters were added to the same method to accomodate different usecases.

For example: sometimes I needed the data hydrated, other times I wanted to work with the actual Doctrine entities from the query's result.

    public function findAll($groupId = null, $hydrate = false) {...}


## Step back

So what exactly are the parameters on the `findAll` function? My answer is: they are usecases where I want to filter the data.

This led to many overloaded repository methods that do too much. But let's pause and take a deep breath: let's not add the `findAllSimple` method yet. There is an easier way to get out of this rabbit hole: *Refactoring!*

### Objective

What I want to do is build a query which suits my usecase. I also don't want to write the same code again and again or get cluttered up in the Eierlegendewollmilchsau-findAll-function.

Here is how to get started!

## Step 1

I want to have a the QueryBuilder-Object. That is fairly simple as I already have the repository class extend the `Doctrine\ORM\EntityRepository`.


    /**
     * @param string $alias
     * @return \Doctrine\ORM\QueryBuilder
     */
    private function getBuilder($alias = 'd')
    {
        return $this->createQueryBuilder($alias);
    }


## Step 2

Next step: pulling all parameter based stuff in their own filterBy*() function

    /**
     * @param \Doctrine\ORM\QueryBuilder $builder
     * @param int                        $groupId
     *
     * @return \Doctrine\ORM\QueryBuilder
     */
    public function filterByGroup(&$builder, $groupId)
    {
        if (null === $groupId) {
            return $this;
        }

        $builder
            ->addWhere('d.groupId = :groupId')
            ->setParameter('groupId', $groupId)
        ;
        return $this;
    }


Once I created these two, I refactored `findAll` to

    public function findAll($groupId = null, $hydrate = false)
    {
        $builder = $this->getBuilder();
        $this
            ->filterByGroup($builder, $groupId)
            ->filterBy*($builder, $param)
        ;
        return $builder;
    }

But `findAll` doesn't take into account that I want _all_ the entries from this result. So in this case, I'll add another usecase method to specialize the query for my second usecase:

    public function getUsersForApi($groupId)
    {
        $builder = $this->findAll();

        $this
            ->filterByGroup($builder, $groupId)
            ->filterBy*($builder, $param)
        ;
        return $builder;
    }


In summary, the pattern I applied to to the repository includes following:

* `get*` for usecases returning the `$builder` i.e. `getXyForMySpecialUsecase($someParam)`
* `filterBy*` adding conditions on the builder returning `$this`
* `findBy*` build in by doctrine for simple queries


## Step 3

Refactor the actions or services where I want to call the repository to


    $groups = $app['orm.ems']['api']
        ->getRepository('\EasyBib\Api\Entity\Group')
        ->getGroupsForApi($groupId)
        ->getQuery()
        ->getSingleResult(AbstractQuery::HYDRATE_ARRAY)
    ;

Executing the query will be in the responsability of the part of the code that actually needs the result.

Last but not least, I refactored:

    public function findAll($groupId = null, $hydrate = false) {...}

to:

    private function findAll()
    {
        return $this
            ->getBuilder()
            ->select(array(/*what ever you standard selects may be*/))
        ;
    }


This ensures that the repository class will grow more healthy and won't end up with helpless clutter.


## Testing

It even becomes trivial to test the queries using this approach: get the query with `$myEntityRepository->getQuery()->getSql()` and test the string for everything you need to be there. This seems to be the most feasible approach for us in order to be able to refactor or extend our existing code base without breaking _anything_.

## Fin

That's all for today!