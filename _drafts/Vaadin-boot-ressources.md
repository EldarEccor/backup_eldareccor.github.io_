@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties implements ResourceLoaderAware {

	private static final String[] SERVLET_RESOURCE_LOCATIONS = { "/" };

	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
			"classpath:/META-INF/resources/", "classpath:/resources/",
			"classpath:/static/", "classpath:/public/" };

		
@Bean
public SimpleUrlHandlerMapping faviconHandlerMapping() {
	SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
	mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
	mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
	faviconRequestHandler()));
	return mapping;
}

@Bean
public ResourceHttpRequestHandler faviconRequestHandler() {
	ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
	requestHandler.setLocations(resolveFaviconLocations());
	return requestHandler;
}

private List<Resource> resolveFaviconLocations() {
	String[] staticLocations = getResourceLocations(
	this.resourceProperties.getStaticLocations());
	List<Resource> locations = new ArrayList<>(staticLocations.length + 1);
	Arrays.stream(staticLocations).map(this.resourceLoader::getResource)
	.forEach(locations::add);
				locations.add(new ClassPathResource("/"));
				return Collections.unmodifiableList(locations);
			}

			@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache()
					.getCachecontrol().toHttpCacheControl();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry
						.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod))
						.setCacheControl(cacheControl));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler(staticPathPattern)
								.addResourceLocations(getResourceLocations(
										this.resourceProperties.getStaticLocations()))
								.setCachePeriod(getSeconds(cachePeriod))
								.setCacheControl(cacheControl));
			}
		}
		
		
Do not use the src/main/webapp directory if your application is packaged as a jar. Although this directory is a common standard, it works only with war packaging, and it is silently ignored by most build tools if you generate a jar.



<!-- Place these in your index.html -->
<link rel="import" href="bower_components/vaadin-lumo-styles/color.html">
<link rel="import" href="bower_components/vaadin-lumo-styles/sizing.html">
<link rel="import" href="bower_components/vaadin-lumo-styles/spacing.html">
<link rel="import" href="bower_components/vaadin-lumo-styles/style.html">
<link rel="import" href="bower_components/vaadin-lumo-styles/typography.html">

<!-- Include the main style modules in global scope (index.html) -->
<custom-style>
  <style include="lumo-color lumo-typography"></style>
</custom-style>