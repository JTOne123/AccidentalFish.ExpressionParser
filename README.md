# AccidentalFish.ExpressionParser

Please note this project is work in progress. It is functional and the API is largely stable but you may trip over work in progress. In particular 
the following are still under development:

	Functions
	String support

## Introduction

This project makes use of Dijkstra's shunting yard algorithm to take an expression expressed as a string (e.g. "5+9*2") and
convert it to an abstract syntax tree that can be used for further purposes. I've spun this code out of another project where
I am using it to:

* Generate terms for Elastic Search queries
* Generate query clauses for Cosmos DB queries
* Generate SQL sub queries (in a scenario where it was not possible to use LINQ)

I've also bundled with this a simple converter that takes the expression tree, converts it to a LINQ expression tree, and compiles an
appropriate lambda. Numerous solutions exist for doing this already (Flee, DynamincLinq) and so this was largely written to facilitate
testing via expression evaluation and just, to be honest, for interest.

## Basic Usage

The simplest way to build an expression tree is to first add the parser package via NuGet:

    Install-Package AccidentalFish.ExpressionParser

After doing so a static method called ParseExpression on the ExpressionFactory class allows an
expression expressed as a string to be parsed into a syntax tree:

    ExpressionNode rootNode = ExpressionFactory.Parse("5+9-3");

This will return the root node of a syntax tree representing the expression. To enumerate each node in the
tree a visitor implementation is provided that will walk the tree as an IEnumerable as shown below:

    foreach(ExpressionNode node in new ExpressionTreeVisitor(rootNode)) {
		Console.WriteLine($"Expression node of type {node.GetType().Name}");
	}

More information on the expression nodes can be found below. To build the tree into a LINQ expression, compile, and
evaluate it first add the LINQ package from NuGet:

    Install-Package AccidentalFish.ExpressionParser.Linq

Now, continuing with our example, the following code will build and compile the expression and display the output:

    Expression expression = LinqExpressionTreeBuilder.Build(rootNode);
	Expression<Func<int>> lambda = Expression.Lambda<Func<int>>(expression);
	Func<int> func = lambda.Compile();
	int result = func();
	Console.WriteLine($"The expression evaluated as {result}");

Additional samples can be found in the tests.

## Using with an IoC Container

The framework supports use through an IoC pattern and appropriate container. To do so
each of the NuGet packages provides a "Dependencies.Register" method (i.e. a static class called Dependencies is exposed that
provides a Register method). This method takes a parameter of type Action<Type, Type> into which you should pass a method that
allows registration of an interface in your container. For example:

    AccidentalFish.ExpressionParser.Dependencies.Register((interface, impl) => myContainer.Register(interface, impl));
	AccidentalFish.ExpressionParser.Linq.Dependencies.Register((interface, impl) => myContainer.Register(interface, impl));
	IExpressionFactory factory = myContainer.Resolve<IExpressionFactory>();
	ILinqExpressionTreeBuilder linqBuilder = myContainer.Resolve<ILinqExpressionTreeBuilder>();
	ExpressionNode rootNode = factory.Parse("5+9+3");
	linqBuilder.Build(rootNode);

## Expression Nodes

All expression nodes ultimately deriver from the abstract ExpressionNode class but are (currently) sub categorised into:

* Operators (deriving from OperatorNode) - addition, subtraction, equality etc.
* Values (deriving from ValueNode) - literal values and variables
* Functions (deriving from FunctionNode) - pow, sin etc.
* Structural (deriving from StructuralNode) - break up expressions and may impact the order of execution

### Operators

|Operator|Type|
|--------|----|
|Addition|AdditionNode|
|Conditional And|ConditionalAndNode|
|Conditional Or|ConditionalOrNode|
|Division|DivisionNode|
|Equality Test|EqualNode|
|Greater Than or Equal To|GreaterThanEqualNode|
|Greater Than|GreaterThanNode|
|Less Than or Equal To|LessThanEqualNode|
|Multiplication|MultiplicationNode|
|Negation|NegateNode|
|Not|NotNode|
|Subtraction|SubstractionNode|

Operators are of one of two subtypes, either a binary node wit left and right associated nodes (for example the addition node)
or a unary node with a single associated now (for example a negation node). They are derived from BinaryOperatorNode and UnaryOperatorNode
respectively.

### Values

Values evaluate to either a double, an integer or a string and can be supplied as a literal within the expression or as a variable (a string
preceded by the @ symbol). The example below demonstrates both:

    5+3*@myvar

Variables allow for the values to be retrieved at the time of evaluation or can be useful as placeholders when the expression tree is being used
to translate into a different format.

### Functions

Function nodes derive from the FunctionNode class and when expressed in an expression string take the typical form of:

    functionName(param1,param2)

Built in functions include:

_work in progress_




## Parsing Approach

## Adding Custom Node Types

## Tests

Just a quick note about the tests - they're a mix of unit and integration developed in a vaguely TDD sense and in a manner
that helped me get from start to something working as quickly as possible.