# Units of Measure (F#)

Floating point and signed integer values in F# can have associated units of measure, which are typically used to indicate length, volume, mass, and so on. By using quantities with units, you enable the compiler to verify that arithmetic relationships have the correct units, which helps prevent programming errors.


## CAPS_SYNTAX_MD

```
[<Measure>] type unit-name [ = measure ]
```

## CAPS_REMARKS_MD
The previous syntax defines *unit-name* as a unit of measure. The optional part is used to define a new measure in terms of previously defined units. For example, the following line defines the measure **cm** (centimeter).


```f#
[<Measure>] type cm
```
The following line defines the measure **ml** (milliliter) as a cubic centimeter (**cm^3**).


```f#
[<Measure>] type ml = cm^3
```
In the previous syntax, *measure* is a formula that involves units. In formulas that involve units, integral powers are supported (positive and negative), spaces between units indicate a product of the two units, **&#42;** also indicates a product of units, and **/** indicates a quotient of units. For a reciprocal unit, you can either use a negative integer power or a **/** that indicates a separation between the numerator and denominator of a unit formula. Multiple units in the denominator should be surrounded by parentheses. Units separated by spaces after a **/** are interpreted as being part of the denominator, but any units following a **&#42;** are interpreted as being part of the numerator.

You can use 1 in unit expressions, either alone to indicate a dimensionless quantity, or together with other units, such as in the numerator. For example, the units for a rate would be written as **1/s**, where **s** indicates seconds. Parentheses are not used in unit formulas. You do not specify numeric conversion constants in the unit formulas; however, you can define conversion constants with units separately and use them in unit-checked computations.

Unit formulas that mean the same thing can be written in various equivalent ways. Therefore, the compiler converts unit formulas into a consistent form, which converts negative powers to reciprocals, groups units into a single numerator and a denominator, and alphabetizes the units in the numerator and denominator.

For example, the unit formulas **kg m s^-2** and **m /s s &#42; kg** are both converted to **kg m/s^2**.

You use units of measure in floating point expressions. Using floating point numbers together with associated units of measure adds another level of type safety and helps avoid the unit mismatch errors that can occur in formulas when you use weakly typed floating point numbers. If you write a floating point expression that uses units, the units in the expression must match.

You can annotate literals with a unit formula in angle brackets, as shown in the following examples.


```f#
1.0<cm>
55.0<miles/hour>
```
You do not put a space between the number and the angle bracket; however, you can include a literal suffix such as **f**, as in the following example.


```f#
// The f indicates single-precision floating point.
55.0f<miles/hour>
```
Such an annotation changes the type of the literal from its primitive type (such as **float**) to a dimensioned type, such as **float&lt;cm&gt;** or, in this case, **float&lt;miles/hour&gt;**. A unit annotation of **&lt;1&gt;** indicates a dimensionless quantity, and its type is equivalent to the primitive type without a unit parameter.

The type of a unit of measure is a floating point or signed integral type together with an extra unit annotation, indicated in brackets. Thus, when you write the type of a conversion from **g** (grams) to **kg** (kilograms), you describe the types as follows.


```f#
let convertg2kg (x : float<g>) = x / 1000.0<g/kg>
```
Units of measure are used for compile-time unit checking but are not persisted in the run-time environment. Therefore, they do not affect performance.

Units of measure can be applied to any type, not just floating point types; however, only floating point types, signed integral types, and decimal types support dimensioned quantities. Therefore, it only makes sense to use units of measure on the primitive types and on aggregates that contain these primitive types.

The following example illustrates the use of units of measure.

```

 // Mass, grams.
 [<Measure>] type g
 // Mass, kilograms.
 [<Measure>] type kg
 // Weight, pounds.
 [<Measure>] type lb 

 // Distance, meters. 
 [<Measure>] type m
 // Distance, cm
 [<Measure>] type cm

 // Distance, inches.
 [<Measure>] type inch
 // Distance, feet
 [<Measure>] type ft

 // Time, seconds.
 [<Measure>] type s

 // Force, Newtons.
 [<Measure>] type N = kg m / s 

 // Pressure, bar.
 [<Measure>] type bar 
 // Pressure, Pascals
 [<Measure>] type Pa = N / m^2 

 // Volume, milliliters.
 [<Measure>] type ml 
 // Volume, liters.
 [<Measure>] type L

 // Define conversion constants.
 let gramsPerKilogram : float<g kg^-1> = 1000.0<g/kg>
 let cmPerMeter : float<cm/m> = 100.0<cm/m>
 let cmPerInch : float<cm/inch> = 2.54<cm/inch>

 let mlPerCubicCentimeter : float<ml/cm^3> = 1.0<ml/cm^3>
 let mlPerLiter : float<ml/L> = 1000.0<ml/L>

 // Define conversion functions.
 let convertGramsToKilograms (x : float<g>) = x / gramsPerKilogram
 let convertCentimetersToInches (x : float<cm>) = x / cmPerInch
```

    The following code example illustrates how to convert from a dimensionless floating point number to a dimensioned floating point value. You just multiply by 1.0, applying the dimensions to the 1.0. You can abstract this into a function like **degreesFahrenheit**.

Also, when you pass dimensioned values to functions that expect dimensionless floating point numbers, you must cancel out the units or cast to **float** by using the **float** operator. In this example, you divide by **1.0&lt;degC&gt;** for the arguments to **printf** because **printf** expects dimensionless quantities.

```

 [<Measure>] type degC // temperature, Celsius/Centigrade
 [<Measure>] type degF // temperature, Fahrenheit

 let convertCtoF ( temp : float<degC> ) = 9.0<degF> / 5.0<degC> * temp + 32.0<degF>
 let convertFtoC ( temp: float<degF> ) = 5.0<degC> / 9.0<degF> * ( temp - 32.0<degF>)

 // Define conversion functions from dimensionless floating point values.
 let degreesFahrenheit temp = temp * 1.0<degF>
 let degreesCelsius temp = temp * 1.0<degC>

 printfn "Enter a temperature in degrees Fahrenheit."
 let input = System.Console.ReadLine()
 let mutable floatValue = 0.
 if System.Double.TryParse(input, &floatValue)
    then 
       printfn "That temperature in Celsius is %8.2f degrees C." ((convertFtoC (degreesFahrenheit floatValue))/(1.0<degC>))
    else
       printfn "Error parsing input."
```

    The following example session shows the outputs from and inputs to this code.


```
Enter a temperature in degrees Fahrenheit.
90
That temperature in degrees Celsius is    32.22.
```

## Using Generic Units
You can write generic functions that operate on data that has an associated unit of measure. You do this by specifying a type together with a generic unit as a type parameter, as shown in the following code example.

```

 // Distance, meters. 
 [<Measure>] type m 
 // Time, seconds. 
 [<Measure>] type s

 let genericSumUnits ( x : float<'u>) (y: float<'u>) = x + y

 let v1 = 3.1<m/s>
 let v2 = 2.7<m/s>
 let x1 = 1.2<m>
 let t1 = 1.0<s>

 // OK: a function that has unit consistency checking.
 let result1 = genericSumUnits v1 v2
 // Error reported: mismatched units.
 // Uncomment to see error.
 // let result2 = genericSumUnits v1 x1
```

    
## Creating Aggregate Types with Generic Units
The following code shows how to create an aggregate type that consists of individual floating point values that have units that are generic. This enables a single type to be created that works with a variety of units. Also, generic units preserve type safety by ensuring that a generic type that has one set of units is a different type than the same generic type with a different set of units. The basis of this technique is that the **Measure** attribute can be applied to the type parameter.

```

  // Distance, meters.
 [<Measure>] type m 
 // Time, seconds. 
 [<Measure>] type s 

 // Define a vector together with a measure type parameter.
 // Note the attribute applied to the type parameter.
 type vector3D<[<Measure>] 'u> = { x : float<'u>; y : float<'u>; z : float<'u>}

 // Create instances that have two different measures.
 // Create a position vector.
 let xvec : vector3D<m> = { x = 0.0<m>; y = 0.0<m>; z = 0.0<m> }
 // Create a velocity vector.
 let v1vec : vector3D<m/s> = { x = 1.0<m/s>; y = -1.0<m/s>; z = 0.0<m/s> }
```

    
## Units at Runtime
Units of measure are used for static type checking. When floating point values are compiled, the units of measure are eliminated, so the units are lost at run time. Therefore, any attempt to implement functionality that depends on checking the units at run time is not possible. For example, implementing a **ToString** function to print out the units is not possible.


## Conversions
To convert a type that has units (for example, **float&lt;'u&gt;**) to a type that does not have units, you can use the standard conversion function. For example, you can use **float** to convert to a **float** value that does not have units, as shown in the following code.

```

 [<Measure>]
 type cm
 let length = 12.0<cm>
 let x = float length
```

    To convert a unitless value to a value that has units, you can multiply by a 1 or 1.0 value that is annotated with the appropriate units. However, for writing interoperability layers, there are also some explicit functions that you can use to convert unitless values to values with units. These are in the [Microsoft.FSharp.Core.LanguagePrimitives](http://msdn.microsoft.com/en-us/library/69d08ac5-5d51-4c20-bf1e-850fd312ece3) module. For example, to convert from a unitless **float** to a **float&lt;cm&gt;**, use [FloatWithMeasure](http://msdn.microsoft.com/en-us/library/69520bc7-d67b-46b8-9004-7cac9646b8d9), as shown in the following code.

```

 open Microsoft.FSharp.Core
 let height:float<cm> = LanguagePrimitives.FloatWithMeasure x
```

    
## Units of Measure in the F# Power Pack
A unit library is available in the F# PowerPack. The unit library includes SI units and physical constants.


## See Also
[F&#35; Language Reference](F%23+Language+Reference.md)
