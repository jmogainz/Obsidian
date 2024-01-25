`skinparam linetype polyline`
 `skinparam linetype ortho`
 `skinparam classAttributeIconSize 0`

Advanced
Pure Abstract or Interface
Composition Objects
Association Class
Association Qualifiers
 - Used to resolve ambiguities in an association where multiple objects are associated with a single object, and you need a way to distinguish among these objects.

##### Relationships
Generalization
- Is type of (do not think it HAS to be inheritance...even though it kind of is...but may be impossible...in actual code implementation....it is a modeling paradigm not a design implementation)
Composition (SpaceType in ISpace)
Directed Composition
Aggregation (ISpace in IObservationHandler)
Directed Aggregation
General Association (Counter which has a countLeaves() method to the Leaf class, can't think of many situations like this)
Dependency (Method uses in return type, body, or param)

Multi-Inheritance vs Composition Technique

{overlapping} or {disjoint} - overlapping cannot occur in c++....kind of need to switch to composition or implement interface where some other class could inherit traits of both
- A parent cant be two children simultatneously

Constraints or properites {}
- **Stereotype Approach**: This is more formal and is particularly useful if you want to establish a clear, standardized pattern within your UML diagrams. It's akin to creating a new "type" of class within your modeling language.
- **Property Approach**: This is more informal and flexible. It's useful for adding notes or metadata without the need to define new stereotypes.




