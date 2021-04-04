Here are some simple rules that if we follow, we will have much less trouble with NullPointerException.


# The rules

### 1.) All public methods perform a null check

### 2.) If a return value of a method may be missing, give an Optional back

```java
Optional<BillingInfo> getBillingInfo(Long id)
{
  if (...)
  {
    return Optional.ofNullable(...);
  }
  
  return Optional.empty();
}
```

### 3.) If a method returns a collection, never return null, return an empty collection instead
```java
List<BillingInfo> getBillingInfos()
{
  List<BillingInfo> retval = new ArrayList();
  ...
  return retval;
}

OR 

List<BillingInfo> getBillingInfos()
{
  if (...)
  {
    ...
    return ...
  }
  
  return Collections.emptyList();
}
```

### 4.) Prefer using wrapper classes as input parameters over primitive types
Consider the following code:
```
Long getId()
{
  ...
}

String getFormattedId(long id)
{
  return String.valueOf(id);
}

void doSomething()
{
  String formattedId = getFormattedId(getId());
}
```
Whenever the method `getId()` returns null, the call of `getFormattedId()` will throw an NPE.

The fool proof solution is as follows:
```
String getFormattedId(Long id)
{
  if (id == null)
  {
    return "";
  }
  
  return String.valueOf(id);
}
```

### 5.) Prefer returning primitive types over wrapper classes
Because primitive types may not be null, chances are higher, that we will not run into an NPE.

### 6.) Pre-instantiate sub-structures
```
private final List<String> labels = new ArrayList<>();
private final BillingInfo billingInfo = new BillingInfo();
```

### 7.) Prefer final properties
Because they will be initialized for sure.

### 8.) Use StringUtils to check if a String is empty or for comparison

```
import org.apache.commons.lang3.StringUtils;

void printMe(String text)
{
  if (StringUtils.isNotBlank(text))
  {
    ...
  }
}
```
It is easier than checking for null, like this:

```
void printMe(String text)
{
  if (text != null && !text.isBlank())
  {
    ...
  }
}
```

### 9.) Chain of calls
We all know structures like:

`getBillingInfo().getContract().getSignee().getPersonalNumber();`

Though it contradicts the Low of Demeter, sometimes we cannot avoid. My rule is as follows: if there are maximal 3 methods in the chain, than use if to check for null. 

```
if (getBillingInfo() != null && getBillingInfo().getContract() != null)
{
  Person = getBillingInfo().getContract().getSignee();  
}
```
Otherwise use Optional.ofNullable().

```
Long personalNumber = Optional.ofNullable(getBillingInfo())
                        .map(billingInfo -> billingInfo.getContract())
                        .map(contract -> contract.getSignee())
                        .map(signee -> signee.getPersonalNumber())
                        .orElse(null);
```
If any of the calls return null, the final result be null. 
