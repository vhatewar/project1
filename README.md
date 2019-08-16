# project1

/*** following is the list of HTTPHeaders

// For HTTP Headers - create MAP
package common ( 

object HTTPHeaders {

	val HttpHeaders_Default = Map(
		"User-Agent" -> "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36",
		"Upgrade-Insecure-Requests" -> "1",
		"Accept-Language" -> "en-GB,en-US;q=0.8,en;q=0.6",
		"Accept" -> "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
		"x-test" -> "pt"
	)

	val HttpHeadersAPI_Default = Map(
		"User-Agent" -> "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36",
		"Accept-Language" -> "en-GB,en-US;q=0.8,en;q=0.6",
		"Content-Type" -> "application/json",
	//	"accept-encoding" -> "gzip, deflate, br,",
		"Accept" -> "application/json, text/plain, */*",
		"x-test" -> "pt"
	)

	val HttpHeadersAjax_Default = Map(
		"User-Agent" -> "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36",
		"Accept-Language" -> "en-GB,en-US;q=0.8,en;q=0.6",
		"X-Requested-With" -> "XMLHttpRequest",
		"Accept" -> "*/*",
		"x-test" -> "pt"
	)

	val HttpHeaders_MlcWorkTracker_Ajax = Map(
		"User-Agent" -> "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36",
		"Accept-Language" -> "en-GB,en-US;q=0.8,en;q=0.6",
		"Accept" -> "application/json, text/javascript, */*"
	)


	val HttpHeaders_Chrome = Map(
		"User-Agent" -> "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36",
		"Accept-Language" -> "en-GB,en-US;q=0.8,en;q=0.6",
		"Accept" -> "application/json, text/plain, */*"
	)

}


//Configration file 

package common


import com.typesafe.config.ConfigFactory
import org.slf4j.LoggerFactory

import scala.util.Properties

object Configuration {
  val logger = LoggerFactory.getLogger(this.getClass)
  val defaultConfigurationFile = "application.conf"
  val configFileSystemProperty = Properties.propOrElse("configuration", defaultConfigurationFile)
  val configFile = Properties.propOrElse("configuration1", defaultConfigurationFile) match {
    case configFileName if configFileName == defaultConfigurationFile => ConfigFactory.load()
    case configFileName => ConfigFactory.load(configFileName)
  }

  val applicationUnderTest = System.getProperty("gatling.application")
  val performanceTest = System.getProperty("gatling.performance.test")
  val environment = System.getProperty("gatling.performance.environment")
  val proxyUrl = System.getProperty("gatling.performance.proxyUrl")
  val proxyPort = System.getProperty("gatling.performance.proxyPort")

  val users = configFile.getInt(s"""gatling.$applicationUnderTest.scenario.$performanceTest.users""")
  val duration = configFile.getInt(s"""gatling.$applicationUnderTest.scenario.$performanceTest.duration""")
  val ramp = configFile.getLong(s"""gatling.$applicationUnderTest.scenario.$performanceTest.ramp""")
  val thinkTime = configFile.getInt(s"""gatling.$applicationUnderTest.scenario.$performanceTest.thinkTime""")
  val paceFrom = configFile.getInt(s"""gatling.$applicationUnderTest.scenario.$performanceTest.paceFrom""")
  val paceTo = configFile.getInt(s"""gatling.$applicationUnderTest.scenario.$performanceTest.paceTo""")

  val baseURL = configFile.getString(s"""gatling.$applicationUnderTest.environment.$environment.baseURL""")
  val hostname = configFile.getString(s"""gatling.$applicationUnderTest.environment.$environment.hostname""")
  val port = configFile.getString(s"""gatling.$applicationUnderTest.environment.$environment.port""")
  val proxyRequired = configFile.getString(s"""gatling.$applicationUnderTest.environment.$environment.proxyRequired""")

  val executionDuration = duration + ramp + ramp
  println(
    s"""

================================================================================
---- User Set Simulation Configuration -> $configFileSystemProperty ($applicationUnderTest)
================================================================================

\tEnvironment: $applicationUnderTest ($environment)
\thostname: $hostname
\tbase URL: $baseURL
\tports: $port
\tTest Type: $performanceTest
\tUsers: $users users\t\t\t/ pacing ($paceFrom,$paceTo)
\tramp up time: $ramp\t\t\t/ duration: $duration
\tTest will execute for approximately $executionDuration seconds

    """)
}

//HTTPProtocol 

package common

import io.gatling.core.Predef._
import io.gatling.http.Predef._


object HTTPProtocol {

  val whiteList = WhiteList(Configuration.baseURL ++ """.*""")
  val blackList = BlackList(""".*login-bg-banner.jpg""")

	val httpProtocol = http
    .baseURL(Configuration.baseURL ++ ":" ++ Configuration.port )
    .virtualHost(Configuration.hostname)
		.acceptEncodingHeader("gzip, deflate, sdch")
    .warmUp(Configuration.baseURL + "/")
    //.inferHtmlResources(whiteList, blackList)

  val httpProtocol_WithProxy = http
    .baseURL(Configuration.baseURL ++ ":" ++ Configuration.port )
    //.baseURL(Configuration.baseURL)
    .virtualHost(Configuration.hostname)
    .acceptEncodingHeader("gzip, deflate, sdch")
    .warmUp(Configuration.baseURL + "/")
    //.inferHtmlResources(whiteList, blackList)
    //.proxy(Proxy("sparpxyapp.aur.national.com.au", 8080).httpsPort(8080))
    .proxy(Proxy(Configuration.proxyUrl, Configuration.proxyPort.toInt).httpsPort(Configuration.proxyPort.toInt))




  

  }
