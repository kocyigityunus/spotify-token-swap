# spotify-token-swap
spotify token swap service in c# with asp.net mvc controller

Configuration : you need to setup **clientId**, **clientSecret** and **callbackUri** from web.config file.

```cs
using System;
using System.Collections.Generic;
using System.Configuration;
using System.IO;
using System.Linq;
using System.Net;
using System.Text;
using System.Web;
using System.Web.Mvc;

namespace SpotifyTokenSwap.Controllers
{
    public class SpotifyController : Controller
    {

        string AUTH_HEADER = "Basic " + Convert.ToBase64String(
            Encoding.UTF8.GetBytes(
                ConfigurationManager.AppSettings["clientId"] + ":" +
                ConfigurationManager.AppSettings["clientSecret"]));

        [HttpPost]
        public ActionResult swap()
        {

            try
            {
                Stream reqBodyStream = Request.InputStream;
                reqBodyStream.Seek(0, System.IO.SeekOrigin.Begin);
                string reqBody = new StreamReader(reqBodyStream).ReadToEnd();

                if (String.IsNullOrWhiteSpace(reqBody))
                {
                    return new HttpStatusCodeResult(550, "unsufficent or deficent request parameters");
                }

                Dictionary<string,string> kv = reqBody.Split(' ')
                    .Select(x => x.Split('='))
                    .Where(x => x.Length == 2)
                    .ToDictionary(x => x.First(), x => x.Last());

                var postData = "grant_type=authorization_code";
                postData += "&redirect_uri=" + ConfigurationManager.AppSettings["callbackUri"];
                postData += "&code=" + kv["code"];

                var data = Encoding.ASCII.GetBytes(postData);

                WebRequest webRequest = WebRequest.Create("https://accounts.spotify.com/api/token");
                webRequest.Method = "POST";
                webRequest.ContentType = "application/x-www-form-urlencoded";
                webRequest.Headers.Set("Authorization", AUTH_HEADER);


                webRequest.ContentLength = data.Length;

                using (var resBodyStream = webRequest.GetRequestStream())
                {
                    resBodyStream.Write(data, 0, data.Length);
                }

                HttpWebResponse response = (HttpWebResponse)webRequest.GetResponse();

                if (response.StatusCode != HttpStatusCode.OK)
                {
                    return new HttpStatusCodeResult(550, "Error came from spotify");
                }

                string resBody = new StreamReader(response.GetResponseStream()).ReadToEnd();
                return Content(resBody, "application/json");
            }
            catch (Exception ex)
            {
                return new HttpStatusCodeResult(550, "Internal Error");

            }

        }

        [HttpPost]
        public ActionResult refresh()
        {

            try
            {
                Stream reqBodyStream = Request.InputStream;
                reqBodyStream.Seek(0, System.IO.SeekOrigin.Begin);
                string reqBody = new StreamReader(reqBodyStream).ReadToEnd();

                if (String.IsNullOrWhiteSpace(reqBody))
                {
                    return new HttpStatusCodeResult(550, "unsufficent or deficent request parameters");
                }

                Dictionary<string, string> kv = reqBody.Split(' ')
                    .Select(x => x.Split('='))
                    .Where(x => x.Length == 2)
                    .ToDictionary(x => x.First(), x => x.Last());

                var postData = "grant_type=refresh_token";
                postData += "&refresh_token=" + kv["refresh_token"];

                var data = Encoding.ASCII.GetBytes(postData);

                WebRequest webRequest = WebRequest.Create("https://accounts.spotify.com/api/token");
                webRequest.Method = "POST";
                webRequest.ContentType = "application/x-www-form-urlencoded";
                webRequest.Headers.Set("Authorization", AUTH_HEADER);
                webRequest.ContentLength = data.Length;

                using (var resBodyStream = webRequest.GetRequestStream())
                {
                    resBodyStream.Write(data, 0, data.Length);
                }

                HttpWebResponse response = (HttpWebResponse)webRequest.GetResponse();

                if (response.StatusCode != HttpStatusCode.OK)
                {
                    return new HttpStatusCodeResult(550, "Error came from spotify");
                }

                string resBody = new StreamReader(response.GetResponseStream()).ReadToEnd();
                return Content(resBody, "application/json");
            }
            catch (Exception ex)
            {
                return new HttpStatusCodeResult(550, "Internal Error");
            }

        }

        public ActionResult test()
        {
            return Content("Test");
        }

    }
}

```
