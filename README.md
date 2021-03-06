# Backbone.TreeModel

## Usage

###1. Initialize
```javascript
var treeObject = {
	tagname: 'body',
	nodes: [
		{
			id: 'sidebar',
			tagname: 'div',
			width: 300,
			nodes: [
				{ tagname: 'p' },
				{ tagname: 'span' }
			]
		},
		{
			id: 'content',
			tagname: 'div',
			width: 600,
			nodes: [
				{ tagname: 'div' },
				{
					tagname: 'p',
					nodes: [
						{
							tagname: 'anchor',
							nodes: [
								{ tagname: 'span' }
							]
						}
					]
				}
			]
		}
	]
};
var tree = new Backbone.TreeModel(treeObject);

```

Backbone.TreeModel can be extended with a `nodesAttribute` property. Defaults to `nodes`, but can be changed if the nodes exist in a container with a different name.

###2. Traversing Tree
```javascript
tree.get('tagname');                                       // returns "body", `tree` is a backbone model
var nodes = tree.nodes();                                  // `nodes` is a backbone collection
var sidebar = nodes.first();                               // `sidebar` is a backbone model
sidebar.get('width');                                      // 300

tree.find('sidebar').next().id;                            // "content"
tree.find('content').prev().id;                            // "sidebar"

// node hierarchy
(sidebar.parent() === tree)                                // true
(sidebar.root() === tree)                                  // true
tree.isRoot();                                             // true
tree.contains(sidebar);                                    // true
sidebar.contains(tree);                                    // false
```

###3. Special Search
```javascript
var content = tree.find('content');                        // `find` returns unique node
var paragraphNode = tree.findWhere({tagname: 'p'});        // `findWhere` returns first match
var paragraphNodes = tree.where({tagname: 'p'});           // `where` returns all matches

// TreeModel nodes inherit from Backbone Models
content.get('tagname');                                    // "div"
content.where({ tagname: 'p'}).length;                     // 1
tree.where({ tagname: 'p' }).length;                       // 2

// Recursive-where for TreeCollection
var nodes = tree.nodes();
nodes.where({tagname: 'div'}).length;                      // 2
nodes.where({tagname: 'div'}, {deep: true}).length;        // 3

// Special mixed TreeModel node Array
var specialArray = tree.where({tagname: 'div'});           // Array with standard methods (push/pop/splice/etc.)
specialArray.length;                                       // 3
var array2 = specialArray.where({tagname: 'span'});        // has where method and returns special array
array2.length;                                             // 2
```

###4. Adding Nodes
```javascript
var sidebar = tree.find('sidebar');

// add single node
sidebar.where({tagname: 'p'}).length;                      // 1
sidebar.add({tagname: 'p'});                               // adds object to sidebar node
sidebar.where({tagname: 'p'}).length;                      // 2

// add array of objects
sidebar.where({tagname: 'span'}).length;                   // 1
sidebar.add([
  { tagname: 'span'},
  { tagname: 'span'}
]);
sidebar.where({tagname: 'span'}).length;                   // 3

// adding a node to the left of the current node
sidebar.insertBefore({id: 'sidebar_left'});
sidebar.prev().id                                          // "sidebar_left"

// adding a node to the right of the current node
sidebar.insertAfter({id: 'sidebar_right'});
sidebar.next().id                                          // "sidebar_right"

// adding or inserting other nodes in the tree will perform a "move"
var tree = new Backbone.TreeModel(treeObject);             // start over with original data
tree.find('content').add(tree.find('sidebar'));            // add existing node
tree.find('content').nodes().length;                       // 3
tree.find('sidebar').parent() == tree.find('content');     // true
```

###5. Removing Nodes
```javascript
var tree = new Backbone.TreeModel(treeObject);             // start over with original data
tree.find('wrapper').nodes().length;                       // 2
tree.find('sidebar').remove();                             // remove sidebar node
tree.find('wrapper').nodes().length;                       // 1
tree.find('wrapper').nodes().first().id;                   // "content"

var tree = new Backbone.TreeModel(treeObject);             // start over with original data
tree.where({tagname: 'span'}).length;                      // 2
tree.remove({tagname: 'span'}, true);                      // remove first matched span
tree.where({tagname: 'span'}).length;                      // 1

var tree = new Backbone.TreeModel(treeObject);             // start over with original data
tree.where({tagname: 'span'}).length;                      // 2
tree.remove({tagname: 'span'});                            // remove all matched nodes
tree.where({tagname: 'span'}).length;                      // 0
```
