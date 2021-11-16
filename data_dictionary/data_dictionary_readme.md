# Overview 
- [Introduction to Data Models](#introduction-to-data-models)
- [Data Dictionary](#data-dictionary)
   - [Data Dictionary Structure](#data-dictionary-structure)
   - [Node Properties and Examples](#node-properties-and-examples)
   - [Dictionary Changes](#dictionary-changes)
      - [Breaking Changes](#breaking-changes)
      - [Entity Relation Additions](#entity-relation-additions)
      - [Schema Additions](#schema-additions)
      - [Cosmetic Corrections](#cosmetic-corrections)
      - [Testing](#testing)
      - [Versioning](#versioning)


# Introduction to Data Models

Every Gen3 data commons employs a data model/data dictionary, which serves to describe, organize, and harmonize data sets submitted by different users. Data harmonization facilitates cross-project analyses and is thus one of the pillars of the data commons paradigm.

The data model organizes experimental metadata variables, “properties”, into linked categories, “nodes”, through the use of a data dictionary. The data dictionary lists and describes all nodes in the data model, as well as defines and describes the properties in each node.

A data commons is powered by the Gen3 software stack. For more information on Gen3, visit the documentation at https://gen3.org. 

> 🟢 Note: This documentation is aimed towards handling a data dictionary on GitHub. For more thorough details about how Gen3 uses a data model and how the data dictionary should be formatted and created, see https://gen3.org/resources/user/dictionary/ and https://gen3.org/resources/operator/#3-creating-a-new-data-dictionary.

# Data Dictionary

The data dictionary provides the first level of validation for all submitted data.
Written in YAML, JSON schemas define all the individual entities
(nodes) in the data model. Moreover, these schemas define all of the relationships (links)
between the nodes. Finally, the schemas define the valid key-value pairs that can be used to
describe the nodes.

## Data Dictionary Structure

The Data Model covers all of the nodes within the as well as the relationships between
the different types of nodes. All of the nodes in the data model are strongly typed and individually
defined for a specific data type. Doing such allows for faster querying of
the data model as well as providing a clear and concise representation of the data.

Beyond node type, there are also a number of extensions used to further define the nodes within
the data model. Nodes are grouped up into categories that represent broad roles for the node such
as `Administrative`, `Clinical`, or `Biospecimen` (more information [here](https://gen3.org/resources/operator/#3-creating-a-new-data-dictionary)).

Additionally, nodes are defined within their `Program` or `Project` and have descriptions of their use (more information on programs and projects [here](https://gen3.org/resources/user/access-data/#the-data-commons-website)). All nodes also have a series of `systemProperties`; these
properties are those that will be automatically filled by the system unless otherwise defined by
the user. These basic properties define the node itself but still need to be placed into the model.

The model itself is represented as a graph. Within the schema are defined `links`; these links
point from child to parent with Program being the root of the graph. The links also contain a
`backref` that allows for a parent to point back to a child. Other features of the link include a
semantic `label` that describes the relationship between the two nodes, a `multiplicity` property
that describes the numeric relationship from the child to the parent, and a requirement property
to define whether a node must have that link. Taken all together the nodes and links create the
directed graph of the Data Model.

## Node Properties and Examples

Each node contains a series of potential key-value pairs (`properties`) that can be used to
characterize the data they represent. Some properties are categorized as `required`, and if a submission lacks a required property, it cannot be accepted. All properties not designated `required` can be left out of submissions.

The properties have further validation through their entries. For basic string type properties, such as `submitter_id`, any string value will be accepted as a valid entry. Other acceptance criteria can be defined in property definitions. For example, `enum` type properties only allow one of a list of strings, which are enumerated in the property's definition. Regular expressions can also be used to limit string submissions to matching values. For numeric properties, maximum and minimum values can limit valid entries.



## Dictionary Changes

The following is an attempt to layout guidelines for the level of
impact of changes to the dictionary by categorizing them into
**Breaking Changes**, **Entity Relation Additions**, **Schema Additions**,
**Cosmetic Corrections**.

### Breaking Changes

Breaking changes are changes to the dictionary such that previously
allowable data is invalid against the new schema, e.g. a **removal** of
part of the dictionary.

N.B. That not all changes classified here as Breaking Changes are
promised to require a data migration. It is possible that no data
exists in the GDC that is invalidated by the change, e.g. making a
field required that has never been left blank. This should be
confirmed against the corpus of data and the userbase should be
notified of a break in backwards-compatibility.

**Breaking Changes include**:
- Renaming/removing anything that is not a description or comment
  - Removing an entity schema
  - Removing a property's allowed `type`
  - Removing a property's allowed `enum` value
  - Changing an entity's `category`
  - Changing an entity's `unique_keys`
  - Changing an entity's `links`, including `label`, `backref`
  - Removing a property from an entity schema
- Changing existence requirements
  - Adding a property to the `required` list
  - Changing link `required` from `false` to `true`
  - Changing link `multiplicity` from `one_to_many` or `many_to_one` to `one_to_one`
  - Changing link subgroup exclusivity from `false` to `true`


**Handling breaking changes**:

Sometimes it may be best to introduce necessary breaking changes
incrementally. Given you have State A and State B, which are
incomatible, if you can create a State AB that is compatible with
both, you can upgrade to State AB without breaking changes, update
data to be compliant with State B, then upgrade to State B.

1. State A is deployed
2. Upgrade to State AB
3. Update data while State AB is deployed to be valid under State B
4. Upgrade to State B

An example could be: _Introduce required property `color`_:

1. Property `color` does not exist
2. Deploy schema that allows but does not require `color`
3. Add color to all records
4. Deploy schema that requires `color`


### Entity Relation Additions

Additions to the dictionary that create nodes, properties, or add links between nodes are not considered breaking changes, however, they should be carefully considered in context of downstream effects.

**Entity Relation Additions include**:
- Adding a new child node;
- Adding a new property to an existing node;
- Adding a new link between nodes without disrupting existing links.

**Entity Relation downstream effects**:
- Data commons operator will need to update the database schema;
- Users should be notified of additions.


### Schema Additions

The data commons is setup to allow strict additions to properties have minimal impact on existing data.

**Schema Additions include**:
- New properties;
- Less restrictive types for properties (e.g., changing enum or boolean to a string, or a number to integer);
- New allowed `enum` members for properties.

**Schema Addition downstream effects**:
- Users should be notified of additions

### Cosmetic Corrections

Cosmetic corrections are changes that have little to no behavioral
effects.

**Cosmetic Corrections include**:
- Changes to terms
- Changes to documentation
- Schema formatting changes

**Schema Addition downstream effects**:
- No large impacts


### Testing

Commits will automagically be run on TravisCI when a Pull Request is opened.
If you would like to test locally they are run via [dictionaryutils](https://github.com/uc-cdis/dictionaryutils).


### Versioning

The data dictionary should follow [Semantic Versioning](http://semver.org/) by updating the line in setup.py file to `MAJOR.MINOR.PATCH` accordingly:

1. MAJOR: version when you make incompatible API changes: **Breaking Changes**
   - e.g. 1.2.4 -> 2.0.0
2. MINOR: version when you add functionality in a backwards-compatible manner: **Relationship Additions**, **Schema Additions**
   - e.g. 1.2.4 -> 1.3.0
3. PATCH: version when you make backwards-compatible bug fixes: **Cosmetic Corrections**
   - e.g. 1.2.4 -> 1.2.5