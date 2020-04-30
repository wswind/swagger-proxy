https://github.com/domaindrivendev/Swashbuckle.AspNetCore/issues/662

nginxurl:5011/api1 => apiurl:5000

nginx-config/swaggertest.conf:

server {
        listen       5011;
        server_name  _;

        location /api1/ {
            proxy_pass         http://192.168.56.1:5000/;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection keep-alive;
            proxy_set_header   Host $http_host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
			proxy_set_header   X-Forwarded-Prefix api1; #to pass the api name
        }
    }


code:

app.UseSwagger(c =>
{
	c.PreSerializeFilters.Add((swagger, httpReq) =>
	{
		swagger.Servers = new List<OpenApiServer> { new OpenApiServer { Url = $"{httpReq.Scheme}://{httpReq.Host.Value}/{httpReq.Headers["X-Forwarded-Prefix"]}" } };
	});
});

app.UseSwaggerUI(c =>
{
	c.SwaggerEndpoint("v1/swagger.json", "My API V1");
});

