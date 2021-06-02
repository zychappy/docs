消费方启动类加上@EnableFeignClients

注意定义的接口如果不和启动类在同一个包路径下，需要加basePackages

即：@EnableFeignClients(basePackages = "com.study")