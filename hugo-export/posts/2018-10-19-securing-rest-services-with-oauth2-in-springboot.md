---
title: Securing REST services with Oauth2 in SpringBoot
author: El Profe
type: post
date: 2018-10-19T05:24:37+00:00
url: /2018/10/19/securing-rest-services-with-oauth2-in-springboot/
categories:
  - java
  - security
  - seguridad
  - spring boot
  - thymeleaf
tags:
  - java
  - mvc
  - rest
  - security
  - spring boot

---
<div class="entry-content">
  <p>
    <span class="notranslate"> Good, students.</span> <span class="notranslate"> In this post I will explain how we can provide security to REST services in Spring Boot.</span> <span class="notranslate"> The example application is the same as the <a href="http://translate.googleusercontent.com/translate_c?depth=1&hl=es&rurl=translate.google.com&sl=auto&sp=nmt4&tl=en&u=http://www.profesor-p.com/2018/10/17/seguridad-web-en-spring-boot/&xid=17259,15700022,15700124,15700149,15700186,15700191,15700201,15700214&usg=ALkJrhjboJBlu1KcRYGQjoNSCBElN-QvSw">previous WEB security entry</a> (in Spanish), so the source code is in: <a href="https://translate.googleusercontent.com/translate_c?depth=1&hl=es&rurl=translate.google.com&sl=auto&sp=nmt4&tl=en&u=https://github.com/chuchip/OAuthServer&xid=17259,15700022,15700124,15700149,15700186,15700191,15700201,15700214&usg=ALkJrhgJJZk509z4q4oVDPmxqiJdqcz2Uw" target="_blank" rel="noopener noreferrer">https://github.com/chuchip/OAuthServer</a> .</span>
  </p>
  
  <h3>
    <span class="notranslate"> &#8211; Explaining the Oauth2 technology</span>
  </h3>
  
  <p>
    <span class="notranslate"> As I said, we will use the OAuth2 protocol, so the first thing will be to explain how this protocol works.</span>
  </p>
  
  <p>
    <span class="notranslate"> OAuth2 has some variants but I am going to explain what I will use in the program and, for this, I will give you an example so that you understand what we intend to do.</span>
  </p>
  
  <p>
    <span class="notranslate"> I will put a daily scene: Payment with a credit card in a store. In this case there are three partners: The store, the bank and us.</span> <span class="notranslate"> Something similar happens in the Oauth2 protocol.</span> <span class="notranslate"> These are the steps:</span>
  </p>
  
  <ol>
    <li>
      <span class="notranslate"> The client, that is, the buyer, asks the bank for a credit card, so that the bank will give us, verify who we are, and give us a credit depending on the money we have in the account or it tells us not to let&#8217;s waste time ;-).</span> <span class="notranslate"> In the OAuth2 protocol to which it grants the cards, it is called the Authentication Server.</span>
    </li>
    <li>
      <span class="notranslate"> If the bank has given us the card, we can go to the store, ie the web server, and we present the credit card.</span> <span class="notranslate"> The store does not know us anything, but he can ask the bank, through the card reader, if he can trust us and to what extent (the credit balance).</span> <span class="notranslate"> The store are the Resource Server.</span>
    </li>
    <li>
      <span class="notranslate"> The store depending on the money that the bank says we have will allow us to buy some products or others.</span> <span class="notranslate"> In the OAuth2 analogy, the web server will allow us to access some pages or others depending on whether we are very rich, rich, average or poor.</span>
    </li>
  </ol>
  
  <p>
    <span class="notranslate"> As a comment, say, in case you have not noticed, that authentication servers are usually used.</span> <span class="notranslate"> When you go to a web page and ask you to register, but as an option lets you do it through Facebook or Google, you are using this technology.</span> <span class="notranslate"> Google or Facebook becomes the &#8216;bank&#8217; that issues that &#8216;card&#8217;, the web page that asks you to register, will use it to verify that you have &#8216;credit&#8217; and let you enter.</span> <span class="notranslate"> I hope you understand the example ;-).</span>
  </p>
  
  <p>
    <img class="imagen_con_borde alignleft wp-image-413" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-17.png" sizes="(max-width: 646px) 100vw, 646px" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-17.png 940w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-17-300x148.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-17-768x378.png 768w" alt="" width="646" height="318" />
  </p>
  
  <p>
    <span class="notranslate"> Here you can see the website of the newspaper &#8220;El Pais&#8217;, creating an account.</span> <span class="notranslate"> If we use Google or Facebook, the newspaper (the store) will rely on what those authentication providers tell them.</span> <span class="notranslate"> In this case the only thing the website needs is that you have a credit card, regardless of the balance 😉</span>
  </p>
  
  <h3>
    &nbsp;
  </h3>
  
  <h3>
    &nbsp;
  </h3>
</div>

### &nbsp;

&nbsp;

<div class="entry-content">
  <h3>
    <span class="notranslate"> &#8211;</span> <span class="notranslate"> Creating an Authorization Server</span>
  </h3>
  
  <p>
    <span class="notranslate"> It is understood ?.</span> <span class="notranslate"> OK, so let&#8217;s see how to <span id="result_box" class="short_text" lang="en"><span class="">create the bank, the store and everything you need</span></span> 😉</span>
  </p>
  
  <p>
    <span class="notranslate"> First, in our project, we need to have the appropriate dependencies, we will need the starters&nbsp; <strong>Cloud OAuth2, Security and Web</strong></span>
  </p>
  
  <p>
    <img class="imagen_con_borde aligncenter wp-image-402 size-full" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-13.png" sizes="(max-width: 714px) 100vw, 714px" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-13.png 714w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-13-300x149.png 300w" alt="" width="714" height="354" />
  </p>
  
  <p>
    <span class="notranslate"> Well, let&#8217;s start by defining the bank, this is what we do in the class: <strong>AuthorizacionServerConfiguration</strong></span>
  </p>
  
  <pre> <span class="notranslate"> @Configuration</span>
<span class="notranslate"> @EnableAuthorizationServer</span>
<span class="notranslate"> public class AuthorizacionServerConfiguration extends AuthorizationServerConfigurerAdapter {</span>

	  <span class="notranslate"> @Autowired</span>
	  <span class="notranslate"> @Qualifier ("authenticationManagerBean")</span>
	  <span class="notranslate"> private AuthenticationManager authenticationManager;</span>
	  
	  <span class="notranslate"> @Autowired</span>
	  <span class="notranslate"> private TokenStore tokenStore;</span>
<span class="notranslate"> @Override</span>
<span class="notranslate"> public void configure (ClientDetailsServiceConfigurer clients) throws Exception {</span>
<span class="notranslate"> clients.inMemory ()</span>
		    <span class="notranslate"> .withClient ("client")</span>
            <span class="notranslate"> .authorizedGrantTypes ("password", "authorization_code", "refresh_token", "implicit")</span>
            <span class="notranslate"> .authorities ("ROLE_CLIENT", "ROLE_TRUSTED_CLIENT", "USER")</span>
            <span class="notranslate"> .scopes ("read", "write")</span>
            <span class="notranslate"> .autoApprove (true)</span>	        
            <span class="notranslate"> .secret (passwordEncoder (). encode ("password"));</span>          
	<span class="notranslate"> }</span>
	
	 <span class="notranslate"> @Bean</span>
	    <span class="notranslate"> public PasswordEncoder passwordEncoder () {</span>
	        <span class="notranslate"> return new BCryptPasswordEncoder ();</span>
	    <span class="notranslate"> }</span>
	 <span class="notranslate"> @Override</span>
	 <span class="notranslate"> public void configure (AuthorizationServerEndpointsConfigurer endpoints) throws Exception {</span>
	     <span class="notranslate"> endpoints</span>
	             <span class="notranslate"> .authenticationManager (authenticationManager)</span>	        
	             <span class="notranslate"> .tokenStore (tokenStore);</span>
	 <span class="notranslate"> }</span>
	 
	  <span class="notranslate"> @Bean</span>
	  <span class="notranslate"> public TokenStore tokenStore () {</span>
	      <span class="notranslate"> return new InMemoryTokenStore ();</span>
	  <span class="notranslate"> }</span>	
<span class="notranslate"> }</span></pre>
  
  <p>
    <span class="notranslate"> We start the class by entering it as a configuration with the @ <strong>Configuration</strong> label and then use the <strong>@EnableAuthorizationServer</strong> tag to tell Spring to activate the authorization server.</span> <span class="notranslate"> To define the server properties, we specify that our class extends the <strong>AuthorizationServerConfigurerAdapter</strong> , which implements the <strong>AuthorizationServerConfigurerAdapter</strong> interface, so Spring will use this class to parameterize the server.</span>
  </p>
  
  <p>
    <span class="notranslate"> We define an <strong>AuthenticationManager</strong> type bean that Spring provides automatically and that we will collect with the <strong>@Autowired</strong> tag.</span> <span class="notranslate"> We also define a <strong>TokenStore</strong> object <strong>,</strong> but to be able to inject it we must define it, which we do in the <strong>public</strong> function <strong>TokenStore tokenStore ().</strong></span>
  </p>
  
  <p>
    <span class="notranslate"> The <strong>AuthenticationManager</strong>, as I said, is provided by Spring but we will have to configure it ourselves.</span> <span class="notranslate"> Later I will explain how it is done.</span> <span class="notranslate"> The <strong>TokenStore</strong> or Identifier Store is where the identifiers that our authentication server is supplying will be stored, so that when the resource server (the store) asks for the credit on a credit card it can respond to it.</span> <span class="notranslate"> In this case we use the <strong>InMemoryTokenStore</strong> class that will <strong>store</strong> the identifiers in memory.</span> <span class="notranslate"> In a real application we could use a <strong>JdbcTokenStore</strong> to save them in a database, so that if the application goes down the clients do not have to renew their credit cards 😉</span>
  </p>
  
  <p>
    <span class="notranslate"> In the function <strong>configure (ClientDetailsServiceConfigurer clients) </strong>we specify the credentials of the bank, I say of the administrator of authentications,</span> <span class="notranslate">and the services offered.</span> <span class="notranslate"> Yes, in plural, because to access the bank we must have a username and password for each of the services offered.</span> <span class="notranslate"> This is a very important concept: <em>The user and password is from the bank, not the customer,</em> for each service offered by the bank there will be a single authentication, although it may be the same for different services.</span>
  </p>
  
  <p>
    <span class="notranslate"> I will detail the lines:</span>
  </p>
  
  <ul>
    <li>
      <span class="notranslate"><strong>clients.inMemory ()</strong> Specifies that we are going to store the services in memory.</span> <span class="notranslate"> In a &#8216;real&#8217; application we would save it in a database, an LDAP server, etc.</span>
    </li>
    <li>
      <span class="notranslate"><strong>withClient (&#8220;client&#8221;)</strong> Is the user with whom we will identify in the bank.</span> <span class="notranslate"> In this case it will be called &#8216;client&#8217;.</span> <span class="notranslate"> Would it have been better to call him &#8216;user&#8217;?</span>
    </li>
    <li>
      <span class="notranslate"> to <strong>uthorizedGrantTypes (&#8220;password&#8221;, &#8220;authorization_code&#8221;, &#8220;refresh_token&#8221;, &#8220;implicit&#8221;)</strong> .</span> <span class="notranslate"> We specify the services that we are configuring for the defined user, for &#8216; <strong>client</strong> &#8216;.</span> <span class="notranslate"> In our example we will only use the <strong>password</strong> service.</span>
    </li>
    <li>
      <span class="notranslate"><strong>authorities (&#8220;ROLE_CLIENT&#8221;, &#8220;ROLE_TRUSTED_CLIENT&#8221;, &#8220;USER&#8221;).</strong></span> <span class="notranslate"> Specify roles or groups that the service offered has.</span> <span class="notranslate"> We will not use it in our example either, so let&#8217;s let it run for the time being.</span>
    </li>
    <li>
      <span class="notranslate"><strong>scopes (&#8220;read&#8221;, &#8220;write&#8221;).</strong></span> <span class="notranslate"> The scope of the service.</span> <span class="notranslate"> Nor will we use it in our application.</span>
    </li>
    <li>
      <span class="notranslate"><strong>autoApprove (true)</strong></span> <span class="notranslate"> If you must automatically approve the client&#8217;s requests.</span> <span class="notranslate"> We&#8217;ll say yes to make the application simpler.</span>
    </li>
    <li>
      <strong><span class="notranslate"> secret (passwordEncoder (). encode (&#8220;password&#8221;)).</span></strong> <span class="notranslate"> Password of the client.</span> <span class="notranslate"> Note that the <strong>encode</strong> function that we have defined a little below is called, to specify with what type of encryption the password will be saved.</span> <span class="notranslate"> The <strong>encode</strong> function is annotated with the <strong>@Bean</strong> tag because spring, when we supply the password in an HTTP request, will look for a <strong>PasswordEncoder</strong> object to check the validity of the delivered password.</span>
    </li>
  </ul>
  
  <p>
    <span class="notranslate"> And finally we have the function <strong>configure (AuthorizationServerEndpointsConfigurer endpoints)</strong> where we define which authentication controller and which store of identifiers should use the end points.</span> <span class="notranslate"> Clarify that the end points are the URLs where &#8216;we will talk to our bank&#8217;, to request the piggy cards 😉</span>
  </p>
  
  <p>
    <span class="notranslate"> Ok, we have our authentication server created but we still need the way that he knows who we are and puts us in different groups, according to the credentials introduced.</span> <span class="notranslate"> Well, to do this we will use the same class that we use to protect a web page.</span> <span class="notranslate"> If you have read the previous article: <a href="http://translate.googleusercontent.com/translate_c?depth=1&hl=es&rurl=translate.google.com&sl=auto&sp=nmt4&tl=en&u=http://www.profesor-p.com/2018/10/17/seguridad-web-en-spring-boot/&xid=17259,15700022,15700124,15700149,15700186,15700191,15700201,15700214&usg=ALkJrhjboJBlu1KcRYGQjoNSCBElN-QvSw" target="_blank" rel="noopener noreferrer">http://www.profesor-p.com/2018/10/17/seguridad-web-en-spring-boot/</a> (in Spanish) remember that we created a class that inherited from <strong>WebSecurityConfigurerAdapter</strong> , where we <strong>overwrote</strong> the function <strong>UserDetailsService userDetailsService ().</strong></span>
  </p>
  
  <pre> <span class="notranslate"> @EnableWebSecurity</span>
<span class="notranslate"> public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {</span>
 <span class="notranslate"> ....</span>
    <span class="notranslate"> @Bean</span>
    <span class="notranslate"> @Override</span>
    <span class="notranslate"> public UserDetailsService userDetailsService () {</span>
    	
    	<span class="notranslate"> UserDetails user = User.builder (). Username ("user"). Password (passwordEncoder (). Encode ("secret")).</span>
    			<span class="notranslate"> roles ("USER"). build ();</span>
    	<span class="notranslate"> UserDetails userAdmin = User.builder (). Username ("admin"). Password (passwordEncoder (). Encode ("secret")).</span>
    			<span class="notranslate"> roles ("ADMIN"). build ();</span>
        <span class="notranslate"> return new InMemoryUserDetailsManager (user, userAdmin);</span>
    <span class="notranslate"> }</span>
<span class="notranslate"> ....</span>
<span class="notranslate"> }</span></pre>
  
  <p>
    <span class="notranslate"> Well, users with their roles or groups are defined in the same way.</span> <span class="notranslate"> We should have a class that extends <strong>WebSecurityConfigurerAdapter</strong> and define our users.</span>
  </p>
  
  <p>
    <span class="notranslate"> Now and we can check if our authorizations server works.</span> <span class="notranslate"> Let&#8217;s see how, using the excellent <strong>PostMan</strong> program <strong>.</strong></span>
  </p>
  
  <p>
    <span class="notranslate"> To speak with the &#8216;bank&#8217; to request our credentials, and as we have not defined otherwise, we must go to the URI &#8220;/oauth/token&#8221;.</span> <span class="notranslate"> This is one of the end points I spoke about earlier.</span> <span class="notranslate"> There is more but in our example and since we are only going to use the service &#8216;password&#8217; we will not use more.</span>
  </p>
  
  <p>
    <span class="notranslate"> We will use a HTTP request type POST, indicating that we want to use basic validation, we will put the user and password, which will be those of the &#8220;bank&#8221;, in our example: &#8216;client&#8217; and &#8216;password&#8217; respectively.</span>
  </p>
  
  <p>
    <img class="imagen_con_borde aligncenter wp-image-408 size-full" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-14.png" sizes="(max-width: 848px) 100vw, 848px" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-14.png 848w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-14-300x116.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-14-768x297.png 768w" alt="" width="848" height="328" />
  </p>
  
  <p>
    <span class="notranslate"> In the body of the request, in form-url-encoded format we will introduce the service to request, our username and our password.</span>
  </p>
  
  <p>
    <img class="imagen_con_borde aligncenter wp-image-409 size-full" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-15.png" sizes="(max-width: 800px) 100vw, 800px" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-15.png 800w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-15-300x131.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-15-768x335.png 768w" alt="" width="800" height="349" />
  </p>
  
  <p>
    <span class="notranslate"> and we launched the petition, which should get us an exit like this:</span>
  </p>
  
  <p>
    <img class="imagen_con_borde aligncenter wp-image-410 size-full" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-16.png" sizes="(max-width: 650px) 100vw, 650px" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-16.png 650w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-16-300x106.png 300w" alt="" width="650" height="230" />
  </p>
  
  <p>
    <span class="notranslate"> That &#8216;access_token&#8217; &#8221; <strong>8279b6f2-013d-464a-b7da-33fe37ca9afb</strong> &#8221; is our credit card and is the one that we must present to our resource server (the store) in order to see pages (resources) that are not public.</span>
  </p>
  
  <h3>
    <span class="notranslate"> &#8211;</span> <span class="notranslate"> Creating a Resource Server (ResourceServer)</span>
  </h3>
  
  <p>
    <span class="notranslate"> Now that we have our credit card, we will create the store that accepts that card 😉</span>
  </p>
  
  <p>
    <span class="notranslate"> In our example we are going to create the server of resources and of authentication in the same program, with which Spring Boot, will be in charge to do that I trusted a part in another, without having to configure anything.</span> <span class="notranslate"> If, as usual in real life, the resource server is in one place and the authentication server in another, we should indicate to the resource server, which is our &#8216;bank&#8217; and how to talk to it.</span> <span class="notranslate"> But we&#8217;ll leave that for another entry.</span>
  </p>
  
  <p>
    <span class="notranslate"> The only class of the resource server is <strong>ResourceServerConfiguration</strong></span>
  </p>
  
  <pre> <span class="notranslate"> @EnableResourceServer</span>
<span class="notranslate"> @RestController</span>
<span class="notranslate"> public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter</span>
<span class="notranslate"> {</span>
<span class="notranslate"> .....</span>
<span class="notranslate"> }</span></pre>
  
  <p>
    <span class="notranslate"> Observe the @ <strong>EnableResourceServer</strong> annotation that will cause Spring to activate the resource server.</span> <span class="notranslate"> The tag <strong>@RestController</strong> is because in this same class we will have the resources, but they could be perfectly in another class.</span>
  </p>
  
  <p>
    <span class="notranslate"> Finally, note that the class extends <strong>ResourceServerConfigurerAdapter</strong> this is because we are going to overwrite methods of that class to configure our resource server.</span>
  </p>
  
  <p>
    <span class="notranslate"> As I said before, since the authentication and resources server is in the same program, we only have to configure the security of our resource server.</span> <span class="notranslate"> This is done in the function:</span>
  </p>
  
  <pre> <span class="notranslate"> @Override</span>
<span class="notranslate"> public void configure (HttpSecurity http) throws Exception {</span>
<span class="notranslate"> http</span>
<span class="notranslate"> .authorizeRequests (). antMatchers ("/ oauth / token", "/ oauth / authorize **", "/ publishes"). permitAll ();</span>  
<span class="notranslate"> // .anyRequest (). authenticated ();</span> 

	<span class="notranslate"> http.requestMatchers (). antMatchers ("/ private") // Deny access to "/ private"</span>
<span class="notranslate"> .and (). authorizeRequests ()</span>
<span class="notranslate"> .antMatchers ("/ private"). access ("hasRole ('USER')")</span> 
	<span class="notranslate"> .and (). requestMatchers (). antMatchers ("/ admin") // Deny access to "/ admin"</span>
<span class="notranslate"> .and (). authorizeRequests ()</span>
<span class="notranslate"> .antMatchers ("/ admin"). access ("hasRole ('ADMIN')");</span>
<span class="notranslate"> }</span></pre>
  
  <p>
    <span class="notranslate"> In the previous entry when we defined the security on the web, explained a function called <strong>configure (HttpSecurity http)</strong> , how is it much like this?</span> <span class="notranslate"> Well, if it is basically the same, and in fact it receives an HttpSecurity object that we must configure.</span>
  </p>
  
  <p>
    <span class="notranslate"> I explain line to line the sentences:</span>
  </p>
  
  <ul>
    <li>
      <span class="notranslate"><code>&lt;strong>http.authorizeRequests().antMatchers("/oauth/token", "/oauth/authorize**", "/publica").permitAll()&lt;/strong></code> allow all requests to &#8220;/ oauth / token&#8221;, &#8220;/ oauth / authorize ** &#8220;,&#8221; / publishes &#8220;without any validation.</span>
    </li>
    <li>
      <span class="notranslate"><code>&lt;strong>anyRequest().authenticated()&lt;/strong></code> This line is commented, if not all the resources would be accessible only if the user has been validated.</span>
    </li>
    <li>
      <span class="notranslate"><code>&lt;strong>requestMatchers().antMatchers("/privada")&lt;/strong></code> Deny access to the url &#8220;/ private&#8221;</span>
    </li>
    <li>
      <span class="notranslate"><code>&lt;strong>authorizeRequests().antMatchers("/privada").access("hasRole('USER')")&lt;/strong></code> allow access to &#8220;/ private&#8221; if the validated user has the role &#8216;USER&#8217;</span>
    </li>
    <li>
      <span class="notranslate"><code>&lt;strong>requestMatchers().antMatchers("/admin")&lt;/strong></code> Deny access to the url &#8220;/ admin&#8221;</span>
    </li>
    <li>
      <span class="notranslate"><code>&lt;strong>authorizeRequests().antMatchers("/admin").access("hasRole('ADMIN')")&lt;/strong></code> allow access to &#8220;/ admin&#8221; if the validated user has the role &#8216;ADMIN&#8217;</span>
    </li>
  </ul>
  
  <p>
    <span class="notranslate"> Once we have our resource server created we must only create the services which is done with these lines:</span>
  </p>
  
  <pre> <span class="notranslate"> @RequestMapping ("/ publishes")</span>
  <span class="notranslate"> public String publico () {</span>
	   <span class="notranslate"> return "Public Page";</span>
  <span class="notranslate"> }</span>
  <span class="notranslate"> @RequestMapping ("/ private")</span>
  <span class="notranslate"> public String private () {</span>
         <span class="notranslate"> return "Private Page";</span>
  <span class="notranslate"> }</span>
  <span class="notranslate"> @RequestMapping ("/ admin")</span>
  <span class="notranslate"> public String admin () {</span>
	    <span class="notranslate"> return "Administrator Page";</span>
  <span class="notranslate"> }</span></pre>
  
  <p>
    <span class="notranslate"> As you can see, there are 3 basic functions that only return their corresponding Strings.</span>
  </p>
  
  <p>
    <span class="notranslate"> Let&#8217;s see now how validation works.</span>
  </p>
  
  <p>
    <span class="notranslate"> First we check that we can access &#8220;/publica&#8221; without any validation:</span>
  </p>
  
  <p>
    <img class="imagen_con_borde wp-image-415 size-full aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-18.png" sizes="(max-width: 772px) 100vw, 772px" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-18.png 772w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-18-300x173.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-18-768x443.png 768w" alt="" width="772" height="445" />
  </p>
  
  <p>
    <span class="notranslate"> Right.</span> <span class="notranslate"> This works!!</span>
  </p>
  
  <p>
    <span class="notranslate"> If I try to access the page &#8220;/private&#8221; I receive an error &#8220;401 unauthorized&#8221;, which indicates that we do not have permission to see that page, so we will use the token issued by our authorizations server, for the user &#8216;<em>user</em>&#8216;, to see what happens 😉</span>
  </p>
  
  <p>
    <img class="imagen_con_borde wp-image-417 size-full aligncenter" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-19.png" sizes="(max-width: 840px) 100vw, 840px" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-19.png 840w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-19-300x170.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-19-768x436.png 768w" alt="" width="840" height="477" />
  </p>
  
  <p>
    <span class="notranslate"> Come on, if we can see our private page.</span> <span class="notranslate"> Let&#8217;s try the administrator&#8217;s page &#8230;</span>
  </p>
  
  <p>
    <img class="imagen_con_borde aligncenter wp-image-418 size-full" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-20.png" sizes="(max-width: 846px) 100vw, 846px" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-20.png 846w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-20-300x173.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-20-768x444.png 768w" alt="" width="846" height="489" />
  </p>
  
  <p>
    <span class="notranslate"> Right, we can not see it.</span> <span class="notranslate"> So we&#8217;re going to request a new token from the credential administrator, but identify ourselves with the user &#8216;admin&#8217;.</span>
  </p>
  
  <p>
    <span class="notranslate"> The token returned is: &#8220;&#8221; ab205ca7-bb54-4d84-a24d-cad4b7aeab57 &#8220;.</span> <span class="notranslate"> We use it to see what happens:</span>
  </p>
  
  <p>
    <img class="imagen_con_borde aligncenter wp-image-419 size-full" src="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-21.png" sizes="(max-width: 869px) 100vw, 869px" srcset="http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-21.png 869w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-21-300x183.png 300w, http://www.profesor-p.com/wp-content/uploads/2018/10/Captura-21-768x468.png 768w" alt="" width="869" height="529" />
  </p>
  
  <p>
    <span class="notranslate"> Well, that&#8217;s it, we can go shopping safely!</span> <span class="notranslate"> Now we just need to set up the store and have the products 😉</span>
  </p>
  
  <p>
    <span id="result_box" class="" lang="en">And that&#8217;s it. <span class="">See you in the next article, students.</span></span><span class="notranslate"> 🙂</span>
  </p>
</div>