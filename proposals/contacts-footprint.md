# Footprint for VCARD

Tim Berners-Lee  
Origial Date: 2013-03-13
COnverted to markdown version of: 2016/08/26 17:04:56   
[Source](http://www.w3.org/2015/03/vcard-footprint/footprint.html "Permalink to Footprint for VCARD")

Proposed standard for interoperability of client systems in a solid system.

Status: Document practice in the Contact pane, contact exporter, etc

### This is an RDF footprint

Note this a "footprint", which defines both the RDF graphs stored, and also what URLS are generated and which
statements are in which web resource (file).    The idea of linked data is that the reader can "follow its nose" and will find the
necessary files just by lookup up those of books, groups, people and so on which it already knows about, and following linked in the RDF returned.
When *reading* the data, the "follow your nose" principle must be used, and the reader must not assume that the files will in fact have particular URLs.

When *writing* new data, however, the client must adhere to the footprint, so that clients together write data in an orderly manageable way.

It may be that it is convenient at times to make read-only datasets which adhere only to the graph, and not to the footprint.  Clients which use that data should work fine.

* * *

### The URIs generated

Suppose an addressbook is stored using VCARD repository at a base URI $b.

The book itself has uri `$b/index.ttl#this`

A group with cleaned-up name (spaces converted to _ and multiple __ reduced to one) $n has URI $b/Group/$n/index.ttl#this

A person with UUID $u (was: truncated UUID) has URI $b/Person/$u/index.ttl#this

A mugshot person with UUID $u has URI $b/Person/$u/image.png

### Group file

The group membership triple is stored in the group file but not in the person file. (Design choice: this is so that you can publish someone's details without revealing which groups they are in. This could be varied under other social contraints.)

When there is no known webid for a person, then in the group file we have just a simple membership triple using their id $b/Person/$u/index.ttl#this.


        <#this> vcard:hasMember <../Person/$u/index.ttl#this> .


When there IS a known webid $w, then that instead is used in the membership triple, and the internal ID for the person is equated to the webid.


        <#this> vcard:hasMember <$w> .
        <$w>  =  <../Person/$u/index.ttl#this> .


Note that the owl:sameAs triple can be either way around, as = is commutative.


        <#this> vcard:hasMember <$w>
       <../Person/$u/index.ttl#this> = <$w> .


The group file contains a declation that the group is of type vcard:Group, and the vcard:fn (name) of the group is in BOTH the group file and the group index.

### People index file

A **index** file $b/people.ttl contains copies of the vcard:fn and email triples so that just by loading that one can do the very common task of autocompletion user input for names and emails.

### Group index file

A **index** file $b/groups.ttl contains copies of the vcard:fn of the groups so that just by loading that one can do the very common autocompletion user input for names and emails.

These two files can be found from the book itself.

### Book file


        <#this> vcard:nameEmailIndex  ;
            vcard:fn "Bob's Address Book";
            vcard:groupIndex  .



### Card file


          @prefix vcard: .
          @prefix ab: .  # For mac-specific stuff
          @prefix dc: .
          @prefix xsd: .

          <#this>
           a vcard:Individual; vcard:fn "Test Person";
          vcard:hasName [vcard:given-name "Test";
          vcard:family-name "Person";
          ];
          vcard:organization-name "mit";
          vcard:url [ a vcard:webid; vcard:value ],
          [ a vcard:FOAF; vcard:value ];
          ab:ABPersonFlags 0;
          vcard:hasUID ;
          dc:created "2015-11-10T18:25:06+0000"^^xsd:dateTime;
          dc:modified "2015-11-10T18:26:14+0000"^^xsd:dateTime.



* * *


[Tim BL][2]

[2]: http://www.w3.org/People/Berners-Lee/card#i
