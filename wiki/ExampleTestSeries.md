# ExampleTestSeries
*An example series of parameterized JUnit tests that utilize JCombinatorial.*

The following code is given as an example of how to write automated tests in
Java that are reusable and easy to maintain. JUnit's `Parameterized`
test-runner is used for this purpose, along with JCombinatorial's various
parameter factories. A complete, compilable version of the code is included
within the downloadable package.

## Abstract Test
We have a web-site to test. The site has two types of users, four types of
products, two types of coupons, and four payment methods. We want to test with
all of these, as well as with no coupon.

```
/** Represents an example of a parameterized test.
 * This example verifies that a user can place an order through an E-commerce web-site. */
@RunWith(Parameterized.class) // This annotation tells JUnit to run this as a parameterized test.
public abstract class ExampleTest {
	private static int testCaseIndex = 0; // a counter to enumerate the test-cases for logging
	
	// === TEST PARAMETERS ===
	private User user; private Coupon coupon; private Product product; private Payment payment;
	
	/** Constructs a test case to verify that a user can place an order. */
	public ExampleTest(User user, Product product, Coupon coupon, Payment payment) {
		this.user = user; this.product = product; this.coupon = coupon; this.payment = payment;
	}

	/** Returns the recommended values to test for each test parameter.
	 * @return an array of arrays, where each sub-array contains values for one of the test parameters. */
	protected static Object[][] valuesToTestForEachParameter() {
		return new Object[][] {
			new User[] { User.REGISTERED_USER, User.UNREGISTERED_USER },
			new Product[] { Product.TYPICAL, Product.DROP_SHIP, Product.E_BOOK, Product.GIFT_CERTIFICATE },
			new Coupon[] { Coupon.FIVE_DOLLARS_OFF, Coupon.TEN_PERCENT_OFF, null },
			new Payment[] {
				new WireTransferPayment(), new CreditCardPayment(),
				new PayPalPayment(), new GiftCertificatePayment()
			}
		};
	}

	// === THE TEST ===
	/** Runs the test case. */
	@Test public void userCanPurchaseProductWithCouponAndPayment() {
		System.out.printf(
			"%nTest-case %d: user=«%s», product=«%s», coupon=«%s», payment=«%s»%n",
			testCaseIndex++, user, product, coupon, payment
		);

		System.out.println("Preparatory step: Construct an order.");
		final Order order = new Order();
		order.setUser(user);
		order.addProduct(product);
		order.setCoupon(coupon);
		order.addPayment(payment);

		System.out.println("Test step: Place the order.");
		final ECommerceWebSite webSite = new ECommerceWebSite();
		webSite.placeOrder(order);
	}
}
```

Assuming that the `placeOrder` method contains the requisite assertions
throughout its order-placement process, this test is capable of testing the
placement of arbitrary orders.

Note the `@Test` annotation, the `public` modifier, and the
`void` return-type on the
`userCanPurchaseProductWithCouponAndPayment()` method. Also note that it
takes no parameters. (It gets the test-parameters indirectly through the class
constructor.) Without all of these attributes, JUnit would have an
initialization error.

## Concrete Test Implementations

### Smoke Test
```
/** Represents a smoke-test (all-values) version of ExampleTest.
 * Tests a very limited number of combinations of parameter values, but ensures that all values get tested. */
public class ExampleTest_Smoke extends ExampleTest {
	public ExampleTest_Smoke(User user, Product product, Coupon coupon, Payment payment) {
		super(user, product, coupon, payment);
	}

	/** Returns a tiny list of parameter combinations (i.e. test cases).
	 * @return the combination list. */
	@Parameterized.Parameters public static List<Object[]> listOfParameterCombinations() {
		final ParameterFactory parameterFactory = new AllValuesParameterFactory();
		parameterFactory.setValuesToTestForEachParameter(
			valuesToTestForEachParameter() // use the values suggested by the abstract test
		);
		return parameterFactory.createListOfParameterCombinations();
	}
}
```

Note the `@Parameterized.Parameters` annotation and the `public
static` modifiers on the `listOfParameterCombinations()` method.
Without these, JUnit would throw an exception with the message "No public
static parameters method on class ExampleTest_Smoke".

The `AllValuesParameterFactory` creates a list of four parameter combinations.
(Whichever parameter has the greatest number of test values, the number of
combinations is always equal to that number.) Therefore, `ExampleTest_Smoke`
executes four test cases.

This test gives, within a very short time, a rough idea of how functional the
web-site's order-placement feature is. (Assuming that the input values you
provided are representative of all possible values and that the majority of
your bugs can be triggered by single inputs. Analysis is required of the
software under test in order to assess the validity of this assumption.)

### Regression Test
```
/** Represents a regression-test (all-pairs) version of ExampleTest.
 * Pairs each value of each parameter to each value of all other parameters.
 * Far from exhaustive, but hopefully will find most bugs. */
public class ExampleTest_Regression extends ExampleTest {
	// <insert constructor here, identical to the ExampleTest_Smoke constructor>

	/** Returns a relatively small list of parameter combinations (i.e. test cases).
	 * @return the combination list. */
	@Parameterized.Parameters public static List<Object[]> listOfParameterCombinations() {
		final ParameterFactory parameterFactory = new AllPairsParameterFactory();
		// <create combinations here, just as in ExampleTest_Smoke>
	}
}
```

The `AllPairsParameterFactory` creates 17 parameter combinations. (The number of
combinations required is always equal to or slightly greater than the product
of the sizes of the two largest sets of test values. `4 x 4 = 16`) Therefore,
`ExampleTest_Regression` executes 17 test cases.

This test gives, within a reasonably short time, a pretty good idea of how
functional the web-site's order-placement feature is. (Assuming that the input
values you provided are representative of all possible values and that most of
your bugs can be triggered by single inputs or by certain pairs of inputs.
Analysis is required of the software under test in order to assess the validity
of this assumption.)

### Exhaustive Test
```
/** Represents an exhaustive version of ExampleTest.
 * Tests all possible combinations of the parameter values. */
public class ExampleTest_Exhaustive extends ExampleTest {
	// <insert constructor here, identical to the ExampleTest_Smoke constructor>

	/** Returns a huge list of parameter combinations (i.e. test cases).
	 * @return the combination list. */
	@Parameterized.Parameters public static List<Object[]> listOfParameterCombinations() {
		final ParameterFactory parameterFactory = new AllCombinationsParameterFactory();
		// <create combinations here, just as in ExampleTest_Smoke>
	}
}
```

The `AllCombinationsParameterFactory` creates 96 parameter combinations. (The
number of combinations required is the product of the sizes of all of the value
sets. `2 x 4 x 3 x 4 = 96`) Therefore, `ExampleTest_Exhaustive` executes 96 test
cases.

This test may take a very long time, but it gives an excellent idea of how
functional the web-site's order-placement feature is. (Assuming that the input
values you provided are representative of all possible values. Analysis is
required of the software under test in order to assess the validity of this
assumption.)
