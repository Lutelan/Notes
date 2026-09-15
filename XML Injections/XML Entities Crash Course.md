XML which stands for _extensible markup language_, it is a language used for storing and transporting data. Just like HTML it uses nested systems of tags to store data. It doesn't use predefined tags, tags can be given any name required to help label the data they store.

XML entities are a way of representing data within an XML document instead of using the data itself. XML files also have a DTD(document type definition), that can define the structure of the document the type of data values. It is defined in the optional `DOCTYPE` element at the start of the XML document. XML also allows customs entities to be defined within the DTD, as follows
```
<!DOCTYPE foo [ <!ENTITY myentity "my entity value" > ]>
```
The definition ensures that usage of the reference `&myentity` will result in it being replaced by "my entity value".

XML external entities are custom entities which are defined in locations outside the DTD in which they are declared. The declaration of external entities uses the `SYSTEM` keyword and a URL from which the entity shall be loaded.
```
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "http://normal-website.com" > ]>
```
One can use the `file://` protocol to allow usage of files as well.
```
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///path/to/file" > ]>
```