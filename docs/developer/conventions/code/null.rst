Do not return null
------------------

We will be on the lookout for any code that returns **null** when performing a code review.

Stop
^^^^

Exceptions contains valuable informations about what may have gone wrong. Eating all exceptions is especially bad. Catch only the most specific exceptions you are expecting.

Do not eat exceptions::
   
   try {
      do_some_stuff();
   } catch (Exception e) {
      return null; // BAD PRACTICE  - DO NOT RETURN "null"
   }

An alternative is to invokes some convenience unexpectedException method from the org.geotools.util.Logging class.

Log the exception::

   try {
       do_some_stuff();
   }
   catch (SomeSpecificException e){
      LOG.warning("Could not do some stuff: " + e);
      return SOME_EMPTY_OBJECT;
   }

How to stop returning null? Use one of the many "NullObjects" available to you as a stand in. You are expecting the person calling you to do something when you retuned **null** right? A NullObject is set up to capture exactly what you expect them to do.

Here is an example::
   
   if (userSelected == null ){
      // null filter indicating user has not selected anything
      return Filter.EXCLUDE;
   }

To return a “null” list::
  
   try {
       return a_set;
   }
   catch (SomeSpecificException e){
      Logging.unexpectedException(LOG, e);
      return Collections.EMPTY_LIST;
   }

How to return a "null" Expression::
   
   try {
      return a_expression;
   }
   catch (SomeSpecificException e){
      Logging.unexpectedException(LOG, e);
      return Expression.NULL;
   }

How to return a "null" Filter (negative)::
   
   // Often used if it a programmer error encountering a limitation (say SQL encoding)
   try {
       return a_filter;
   }
   catch (SomeSpecificException e){
       Logging.unexpectedException(LOG, e);
       return Expression.EXCLUDES;
   }

How to return a "null" Filter (positive)::
   
   // Often used if it is a "data problem", return too much data rather then none
   try {
       return a_filter;
   }
   catch (SomeSpecificException e){
       Logging.unexpectedException(LOG, e);
       return Expression.INCLUDES;
   }

If you need additional "NullObjects" email the devel list and we will make them happen. We probably should have a NullDataStore and a NullFeatureSource for example ... or throw an Exception.

Drop
^^^^

Drop the pretence that everything is okay, if you are in a situation where you are returning null chances are there is a real problem around ... so throw some kind of Exception. This should be used when you have defined your limitations in the javadocs, and the programmer has neglected to read them.

This may require use of runtime exceptions, or the interface you are implementing may already provide a checked exception for you to throw.

Programming Error::
   
   try {
       do_some_stuff();
   } catch (SomeSpecificException e) {
       return (IOException) new IOException("Could not do some stuff (did you read the javadocs?): "+e).initCause(e);
   }

Roll
^^^^

As a last possible effort; Roll with the flow (but be sure to "log the cause" so client code can notice something is wrong):

Log the Cause::
   
   try {
      do_some_stuff();
   } catch (SomeSpecificException e) {
      Logging.unexpectedException(LOG, e);
      return null; // hope the calling code can figure it out
   }

Be sure to use the warning level (we are still making an assumption) - and client code should know about it). And do not worry about filling up the test case results with garbage; testing code that expected to produce warnings happens all the time (the test case code can change the logging level to "Info" and then set it back).


Document generated by Confluence on Feb 04, 2011 21:47