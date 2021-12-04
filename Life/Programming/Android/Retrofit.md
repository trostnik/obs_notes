By default retrofit return Call\<T\>. To use retrofit we need to define interface that contains all requests then build Retrofit using RetrofitBuilder, than create class, that implement interface(service) using Retrofit.create().

There are all request method annotations, one of which need to be defined for each call: HTTP, GET, POST, PUT, PATCH, DELETE, OPTIONS and HEAD.
\@Path annotation is used to pass path variable to dynamic request: 
			\@GET("group/{id}/users")
			Call<List\<User\>> groupList(@Path("id") int groupId);
\@Query annotation is used pass query parameter to request:
	 		\@GET("group/{id}/users")
			Call\<List\<User\>\> groupList(\@Path("id") int groupId, \@Query("sort") String sort);
\@QueryMap annotation is used to define complex query parameter combination. It is used to pass a lot of query parameters as a single Map object
\@Body annotaion is used to pass complete object to request, instead of deviding it. The object will be converted using a converter, specified in Retrofit. If no converters are used, then only RequestBody can be used.
\@FormUrlEncoded annotation is used to send form-encoded data(data that is encoded in a way http works with form data).  Each key-value pair is annotated with \@Field containing the name and the object providing the value.
\@Multipart annotation is used to send files and requires converter or only RequestBody can be used.
\@Headers annotation before method is used to provide static headers for request. We can use \@Header annotation inside method signature to provide dynamic headers. \@HeaderMap is like \@QueryMap but for complex headers combinations. Best way to provide headers, that are inside every request is to specify them inside OkHttpInterceptor.
