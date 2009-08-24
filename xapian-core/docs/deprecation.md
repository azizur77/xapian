% Deprecation

<!-- This document was originally written by Richard Boulton.

     Copyright (C) 2007 Lemur Consulting Ltd
     Copyright (C) 2007,2008,2009 Olly Betts

  -->

## Introduction

Xapian's API is fairly stable and has been polished piece by piece over time,
but it still occasionally needs to be changed.  This may be because a new
feature has been implemented and the interface needs to allow access to it, but
it may also be required in order to polish a rough edge which has been missed
in earlier versions of Xapian, or simply to reflect an internal change which
requires a modification to the external interface.

We aim to make such changes in a way that allows developers to work against a
stable API, while avoiding the need for the Xapian developers to maintain too
many old historical interface artefacts.  This document describes the process
we use to deprecate old pieces of the API, lists parts of the API which are
currently marked as deprecated, and also describes parts of the API which have
been deprecated for some time, and are now removed from the Xapian library.

It is possible for functions, methods, constants, types or even whole classes
to be deprecated, but to save words this document will often use the term
"features" to refer collectively to any of these types of interface items.


## Deprecation Procedure

### Deprecation markers

At any particular point, some parts of the C++ API will be marked as
"deprecated".  This is indicated with the ``XAPIAN_DEPRECATED`` macro, which
will cause compilers with appropriate support (such as GCC 3.1 or later, and
MSVC 7.0 or later) to emit warning messages about the use of deprecated
features at compile time.

If a feature is marked with one of these markers, you should avoid using it in
new code, and should migrate your code to use a replacement when possible.  The
documentation comments for the feature, or the list at the end
of this file, will describe possible alternatives to the deprecated feature.

### API and ABI compatibility

Releases are given three-part version numbers (e.g. 1.0.2), the three parts
being termed "major" (1), "minor" (0), and "revision" (2).  Releases with
the same major and minor version are termed a "release series".

For Xapian releases 1.0.0 and higher, an even minor version indicates a stable
release series, while an odd minor version indicates a development release
series.

Within a stable release series, we strive to maintain API and ABI forwards
compatibility.  This means that an application written and compiled against
version `X.Y.a` of Xapian should work, without any source changes or need to
recompile, with a later version `X.Y.b`, for all `b` >= `a`.

Stable releases which increase the minor or major version number will usually
change the ABI incompatibly (so that code will need to be recompiled against
the newer release series.  They may also make incompatible API changes,
though we will attempt to do this in a way which makes it reasonably easy to
migrate applications, and document how to do so in this document.

It is possible that a feature may be marked as deprecated within a minor
release series - that is from version `X.Y.c`
onwards, where `c` is not zero.  The API and ABI will not be changed by this
deprecation, since the feature will still be available in the API (though the
change may cause the compiler to emit new warnings when rebuilding code
which uses the now-deprecated feature).

Users should generally be able to expect working code which uses Xapian not to
stop working without reason.  We attempt to codify this in the following
policy, but we reserve the right not to slavishly follow this.  The spirit of
the rule should kept in mind - for example if we discovered a feature which
didn't actually work, making an incompatible API change at the next ABI bump
would be reasonable.

Normally a feature will be supported after being deprecated for an entire
stable release series.  For example, if a feature is deprecated in release
1.2.0, it will be supported for the entire 1.2.x release series, and removed in
development release 1.3.0.  If a feature is deprecated in release 1.2.1, it
will be supported for the 1.2.x *and* 1.4.x stable release series (and of
course the 1.3.x release series in between), and won't be removed until
1.5.0.

### Experimental features

During a development release series (such as the 1.1.x series), some features
may be marked as "experimental".  Such features are liable to change without
going through the normal deprecation procedure.  This includes changing on-disk
formats for data stored by the feature, and breaking API and ABI compatibility
between releases for the feature.  Such features are included in releases to
get wider use and corresponding feedback about them.

### Deprecation in the bindings

When the Xapian API changes, the interface provided by the Xapian bindings will
usually change in step.  In addition, it is sometimes necessary to change the
way in which Xapian is wrapped by bindings - for example, to provide a better
convenience wrapper for iterators in Python.  Again, we aim to ensure that an
application written (and compiled, if the language being bound is a compiled
language) for version `X.Y.a` of Xapian should work without any changes or need
to recompile, with a later version `X.Y.b`, for all `a` <= `b`.

However, the bindings are a little less mature than the core C++ API, so we
don't intend to give the same guarantee that a feature present and not
deprecated in version `X.Y.a` will work in all versions `X+1.Y.b`.  In other
words, we may remove features which have been deprecated without waiting for
an entire release series to pass.

Any planned deprecations will be documented in the list of deprecations and
removed features at the end of this file.


## How to avoid using deprecated features

We recommend taking the following steps to avoid depending on deprecated
features when writing your applications:

 - If at all possible, test compile your project using a compiler which
   supports warnings about deprecated features (such as GCC 3.1 or later), and
   check for such warnings.  Use the -Werror flag to GCC to ensure that you
   don't miss any of them.
 - Check the NEWS file for each new release for details of any new features
   which are deprecated in the release.
 - Check the documentation comments, or the automatically extracted API
   documentation, for each feature you use in your application.  This
   documentation will indicate features which are deprecated, or planned for
   deprecation.
 - For applications which are not written in C++, there is currently no
   equivalent of the ``XAPIAN_DEPRECATED`` macro for the bindings, and thus
   there is no way for the bindings to give a warning if a deprecated feature
   is used.  This would be a nice addition for those languages in which there
   is a reasonable way to give such warnings.  Until such a feature is
   implemented, all application writers using the bindings can do is to check
   the list of deprecated features in each new release, or lookup the features
   they are using in the list at the end of this file.

## Features currently marked for deprecation

### Native C++ API

<!-- Keep table width to <= 126 columns. -->

------------------------------------------------------------------------------------------------------------------------------
Deprecated Remove Feature name                        Upgrade suggestion and comments
---------- ------ ----------------------------------- ------------------------------------------------------------------------
1.1.0      1.3.0  Default second parameter to         The parameter name was ``ascending`` and defaulted to ``true``.  However
                  ``Enquire`` sorting functions.      ascending=false gave what you'd expect the default sort order to be (and
                                                      probably think of as ascending) while ascending=true gave the reverse
                                                      (descending) order.  For sanity, we renamed the parameter to ``reverse``
                                                      and deprecated the default value.  In the more distant future, we'll
                                                      probably add a default again, but of ``false`` instead.
                                                      
                                                      The methods affected are:
                                                      ``Enquire::set_sort_by_value(Xapian::valueno sort_key)``
                                                      ``Enquire::set_sort_by_key(Xapian::Sorter * sorter)``
                                                      ``Enquire::set_sort_by_value_then_relevance(Xapian::valueno sort_key)``
                                                      ``Enquire::set_sort_by_key_then_relevance(Xapian::Sorter * sorter)``
                                                      ``Enquire::set_sort_by_relevance_then_value(Xapian::valueno sort_key)``
                                                      ``Enquire::set_sort_by_relevance_then_key(Xapian::Sorter * sorter)``
------------------------------------------------------------------------------------------------------------------------------

### Bindings

<!-- Keep table width to <= 126 columns. -->

------------------------------------------------------------------------------------------------------------------------------
Deprecated Remove Language Feature name                 Upgrade suggestion and comments
---------- ------ -------- ---------------------------- ----------------------------------------------------------------------
1.0.4      1.3.0  Python   Non-pythonic iterators       Use the pythonic iterators instead.

1.1.0      1.3.0  Python   Stem_get_available_languages Use Stem.get_available_languages instead (static method instead of
                                                        function)
------------------------------------------------------------------------------------------------------------------------------

## Features currently marked as experimental

### Native C++ API

------------------------------------------------------------------------------------------------------------------------------
Name           Details
-------------- ---------------------------------------------------------------------------------------------------------------
Replication    Replication API (in particular, the ReplicationInfo structure) is likely to be changed in future.  Also, the
               format for changesets (both on-disk and over the network) is likely to change (to allow compression of
	       changesets).  See tickets [#347][ticket-347] and [#348][ticket-348].

PostingSource  The PostingSource API is new, and has already been through several redesigns before release.  While we think
	       that we now have a good design, we are concerned that wider use of it will show that the API needs to be
	       adjusted again, so we're marking it as experimental for now.
------------------------------------------------------------------------------------------------------------------------------

[ticket-347]: http://trac.xapian.org/ticket/347
[ticket-348]: http://trac.xapian.org/ticket/348

## Features removed from Xapian

### Native C++ API

<!-- Keep table width to <= 126 columns. -->

------------------------------------------------------------------------------------------------------------------------------
Removed Feature name                        Upgrade suggestion and comments
------- ----------------------------------- ----------------------------------------------------------------------------------
1.0.0   QueryParser::set_stemming_options() Use ``set_stemmer()``, ``set_stemming_strategy()`` and/or ``set_stopper()``
                                            instead. (See the detailed section for more information.)

1.0.0   Enquire::set_sort_forward()         Use ``Enquire::set_docid_order()`` instead; the parameter is ``ASCENDING`` if you
                                            previously used ``set_sort_forward(true)``, and ``DESCENDING`` for
                                            ``set_sort_forward(false)``.

1.0.0   Enquire::set_sorting()              Use ``Enquire::set_sort_by_relevance()``, ``Enquire::set_sort_by_value()``, or
                                            ``Enquire::set_sort_by_value_then_relevance()`` instead. (See the detailed section
                                            for more information.)

1.0.0   Stem::stem_word(word)               Use ``Stem::operator()(word)`` instead.

1.0.0   Auto::open(path)                    Use the ``Database(path)`` constructor instead.

1.0.0   Auto::open(path, action)            Use the ``WritableDatabase(path, action)`` constructor instead.

1.0.0   Query::is_empty()                   Use ``Query::empty()`` instead.

1.0.0   Document::add_term_nopos()          Use ``Document::add_term()`` instead.

1.0.0   Enquire::set_bias()                 No replacement yet implemented.

1.0.0   ExpandDecider::operator()           Return type is now ``bool`` not ``int``.

1.0.0   MatchDecider::operator()            Return type is now ``bool`` not ``int``.

1.0.0   Error::get_type()                   Return type is now ``const char *`` not ``std::string``.  Most existing code
                                            won't need changes, but if it does the simplest fix is to write
                                            ``std::string(e.get_type())`` instead of ``e.get_type()``.

1.0.0   <xapian/output.h>                   Use ``cout << obj.get_description();`` instead of ``cout << obj;``

1.0.0   Several constructors marked         Explicitly create the object type required, for example use
        as explicit.                        ``Xapian::Enquire enq(Xapian::Database(path));`` instead of
                                            ``Xapian::Enquire enq(path);``

1.0.0   QueryParser::parse_query() throwing Catch ``Xapian::QueryParserError`` instead of ``const char *``, and call
        ``const char *`` exception.         ``get_msg()`` on the caught object.  If you need to build with either version,
                                            catch both (you'll need to compile the part which catches ``QueryParserError``
                                            conditionally, since this exception isn't present in the 0.9 release series).

1.1.0   xapian_version_string()             Use ``version_string()`` instead.

1.1.0   xapian_major_version()              Use ``major_version()`` instead.

1.1.0   xapian_minor_version()              Use ``minor_version()`` instead.

1.1.0   xapian_revision()                   Use ``revision()`` instead.

1.1.0   Enquire::include_query_terms        Use ``Enquire::INCLUDE_QUERY_TERMS`` instead.

1.1.0   Enquire::use_exact_termfreq         Use ``Enquire::USE_EXACT_TERMFREQ`` instead.

1.1.0   Error::get_errno()                  Use ``Error::get_error_string()`` instead.

1.1.0   Enquire::register_match_decider()   This method didn't do anything, so just remove calls to it!

1.1.0   Query::Query(Query::op, Query)      This constructor isn't useful for any currently implemented
                                            ``Query::op``.

1.1.0   The Quartz backend                  Use the Flint backend instead.

1.1.0   Quartz::open()                      Use ``Flint::open()`` instead.

1.1.0   quartzcheck                         Use ``xapian-check`` instead.

1.1.0   quartzcompact                       Use ``xapian-compact`` instead.

1.1.0   quartzdump                          Use ``xapian-inspect`` instead.

1.1.0   configure --enable-debug            configure --enable-assertions

1.1.0   configure --enable-debug=full       configure --enable-assertions --ena
ble-log

1.1.0   configure --enable-debug=partial    configure --enable-assertions=parti
al

1.1.0   configure --enable-debug=profile    configure --enable-log=profile

1.1.0   configure --enable-debug-verbose    configure --enable-log

1.1.0   ``Database::positionlist_begin()``  This check is quite expensive, and often you don't care.  If you 
        throwing ``RangeError`` if the      do it's easy to check - just open a ``TermListIterator`` for the 
        term specified doesn't index the    document and use ``skip_to()`` to check if the term is there. 
        document specified. 

1.1.0   ``Database::positionlist_begin()``  This check is quite expensive, and often you don't care.  If you 
        throwing ``DocNotFoundError`` if    do, it's easy to check - just call ``Database::get_document()`` with the 
        the document specified doesn't      specified document ID. 
        exist. 
------------------------------------------------------------------------------------------------------------------------------


### Bindings

<!-- Keep table width to <- 126 columns. -->

------------------------------------------------------------------------------------------------------------------------------
Removed Language Feature name                 Upgrade suggestion and comments
------- -------- ---------------------------- --------------------------------------------------------------------------------
1.0.0   SWIG     Enquire::set_sort_forward()  Use ``Enquire::set_docid_order()`` instead; the parameter is ``ASCENDING`` if
        [^rswg]                               you previously used ``set_sort_forward(true)``, and ``DESCENDING`` for
                                              ``set_sort_forward(false)``.

1.0.0   SWIG     Enquire::set_sorting()       Use ``Enquire::set_sort_by_relevance()``, ``Enquire::set_sort_by_value()`` or
        [^rswg]                               ``Enquire::set_sort_by_value_then_relevance()`` instead. (See the detailed
                                              section for more information.)

1.0.0   SWIG     Auto::open(path)             Use the ``Database(path)`` constructor instead.
        [^rswg]


1.0.0   SWIG     Auto::open(path, action)     Use the ``WritableDatabase(path, action)`` constructor instead.
        [^rswg] 

1.0.0   SWIG     MSet::is_empty()             Use ``MSet::empty()`` instead.
        [^rsw3] 

1.0.0   SWIG     ESet::is_empty()             Use ``ESet::empty()`` instead.
        [^rsw3] 

1.0.0   SWIG     RSet::is_empty()             Use ``RSet::empty()`` instead.
        [^rsw3] 

1.0.0   SWIG     Query::is_empty()            Use ``Query::empty()`` instead.
        [^rsw3] 

1.0.0   SWIG     Document::add_term_nopos()   Use ``Document::add_term()`` instead.
        [^rswg] 

1.0.0   CSharp   ExpandDecider::Apply()       Return type is now ``bool`` instead of ``int``.

1.0.0   CSharp   MatchDecider::Apply()        Return type is now ``bool`` instead of ``int``.

1.0.0   SWIG     Stem::stem_word(word)        Use ``Stem::operator()(word)`` instead. [^callable]
        [^rswg] 

1.1.0   SWIG     xapian_version_string()      Use ``version_string()`` instead.
        [^rswg] 

1.1.0   SWIG     xapian_major_version()       Use ``major_version()`` instead.
        [^rswg] 

1.1.0   SWIG     xapian_minor_version()       Use ``minor_version()`` instead.
        [^rswg] 

1.1.0   SWIG     xapian_revision()            Use ``revision()`` instead.
        [^rswg]

1.1.0   SWIG     ESetIterator::get_termname() Use ``ESetIterator::get_term()`` instead.  This change is intended to
        [^rswg]                               bring the ESet iterators in line with other term iterators, which all
                                              support ``get_term()`` instead of ``get_termname()``.

1.1.0   Python   get_description()            All ``get_description()`` methods have been renamed to ``__str__()``,
                                              so the normal python ``str()`` function can be used.

1.1.0   Python   MSetItem.get_*()             All these methods are deprecated, in favour of properties.
                                              To convert, just change ``msetitem.get_FOO()`` to ``msetitem.FOO``

1.1.0   Python   Enquire.get_matching_terms   Replaced by ``Enquire.matching_terms``, for consistency with
                                              rest of Python API.

1.1.0   SWIG     Error::get_errno()           Use ``Error::get_error_string()`` instead.
        [^rswg]

1.1.0   SWIG     MSet::get_document_id()      Use ``MSet::get_docid()`` instead.
        [^rsw2]
------------------------------------------------------------------------------------------------------------------------------

[^rswg]: This affects all SWIG generated bindings (currently: Python, PHP, Ruby, Tcl8 and CSharp)

[^rsw2]: This affects all SWIG-generated bindings except those for Ruby, support for which was added after the function was deprecated in Xapian-core.

[^rsw3]: This affects all SWIG generated bindings except those for Ruby, which was added after the function was deprecated in Xapian-core, and PHP, where empty is a reserved word (and therefore, the method remains "is_empty").

[^callable]: Python handles this like C++.  Ruby renames it to 'call' (idiomatic Ruby).  PHP renames it to 'apply'.  CSharp to 'Apply' (delegates could probably be used to provide C++-like functor syntax, but that's effort and it seems debatable if it would actually be more natural to a C# programmer).  Tcl8 renames it to 'apply' - need to ask a Tcl type if that's the best solution.

### Enquire::set_sorting()

``Enquire::set_sorting()`` was deprecated at v1.0.0. You should use ``Enquire::set_sort_by_relevance()``, ``Enquire::set_sort_by_value()`` or ``Enquire::set_sort_by_value_then_relevance()`` instead.

 - ``set_sorting(KEY, 1)`` becomes ``set_sort_by_value(KEY)``
 - ``set_sorting(KEY, 1, false) becomes ``set_sort_by_value(KEY)``
 - ``set_sorting(KEY, 1, true)`` becomes ``set_sort_by_value_then_relevance(KEY)``
 - ``set_sorting(ANYTHING, 0) becomes set_sort_by_relevance()``
 - ``set_sorting(Xapian::BAD_VALUENO, ANYTHING)`` becomes ``set_sort_by_relevance()``

### QueryParser::set_stemming_options()

``QueryParser::set_stemming_options()`` was deprecated at v1.0.0. You should use ``set_stemmer()``, ``set_stemming_strategy()`` and/or ``set_stopper()`` instead.

- ``set_stemming_options("")`` becomes ``set_stemming_strategy(Xapian::QueryParser::STEM_NONE)``
- ``set_stemming_options("none")`` becomes ``set_stemming_strategy(Xapian::QueryParser::STEM_NONE)``
- ``set_stemming_options(LANG)`` becomes ``set_stemmer(Xapian::Stem(LANG)`` and ``set_stemming_strategy(Xapian::QueryParser::STEM_SOME)``
- ``set_stemming_options(LANG, false)`` becomes ``set_stemmer(Xapian::Stem(LANG)`` and ``set_stemming_strategy(Xapian::QueryParser::STEM_SOME)``
- ``set_stemming_options(LANG, true)`` becomes ``set_stemmer(Xapian::Stem(LANG)`` and ``set_stemming_strategy(Xapian::QueryParser::STEM_ALL)``

If a third parameter is passed, ``set_stopper(PARAM3)`` and treat the first two parameters as above.

### Omega

<!-- Keep table width to <- 126 columns. -->

------------------------------------------------------------------------------------------------------------------------------
Removed Feature name                        Upgrade suggestion and comments
------- ----------------------------------- ----------------------------------------------------------------------------------
1.0.0   $freqs                              Use ``$map{$queryterms,$_:&nbsp;$nice{$freq{$_}}}`` instead.

1.0.0   scriptindex -u                      ``-u`` was ignored for compatibility with 0.7.5 and earlier, so just remove it.

1.0.0   scriptindex -q                      ``-q`` was ignored for compatibility with 0.6.1 and earlier, so just remove it.

1.1.0   scriptindex index=nopos             Use ``indexnopos`` instead.
------------------------------------------------------------------------------------------------------------------------------