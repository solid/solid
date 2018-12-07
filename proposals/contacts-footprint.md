Tim Berners-Lee
Date: 2013-03-13 last change: $Date: 2016/08/26 17:04:56 $
Status: documening practing in the Contact pane and the contact exporter

# Storing VCARD data in LDP

## The URIs generated

Suppose an addressbook is stored using VCARD repository at a base URI $b.

The book itself has uri $b/book.ttl#this

A group with cleaned-up name (spaces converted to _ and multiple __ reduced to one) $n has URI $b/Group/$n/index.ttl#this

A person with UUID $u (was: truncated UUID) has URI $b/Person/$u/index.ttl#this

A mugshot person with UUID $u has URI $b/Person/$u/image.png

## Group file

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

## People index file

A index file $b/people.ttl contains copies of the vcard:fn and email triples so that just by loading that one can do the very common task of autocompletion user input for names and emails.
Group index file

A index file $b/groups.ttl contains copies of the vcard:fn of the groups so that just by loading that one can do the very common autocompletion user input for names and emails.

These two files can be found from the book itself.

### Book file

    <#this> vcard:nameEmailIndex  <people.ttl>;
        vcard:fn "Bob's Address Book";
        vcard:groupIndex  <groups.ttl>.
        
### Card file

      @prefix vcard: <http://www.w3.org/2006/vcard/ns#>.
      @prefix ab: <http://www.w3.org/ns/pim/ab#>.  # For mac-specific stuff
      @prefix dc: <http://purl.org/dc/elements/1.1/>.
      @prefix xsd: <http://www.w3.org/2001/XMLSchema#>.

      <#this>
       a vcard:Individual; vcard:fn "Test Person";
      vcard:hasName [vcard:given-name "Test";
      vcard:family-name "Person";
      ];
      vcard:organization-name "mit";
      vcard:url [ a vcard:webid; vcard:value <http://example.org/foobar>],
      [ a vcard:FOAF; vcard:value <http://example.org/test>];
      ab:ABPersonFlags 0;
      vcard:hasUID <urn:uuid:82464976-2F1F-457F-9EB1-BFADC309EA8D:ABPerson>;
      dc:created "2015-11-10T18:25:06+0000"^^xsd:dateTime;
      dc:modified "2015-11-10T18:26:14+0000"^^xsd:dateTime.
