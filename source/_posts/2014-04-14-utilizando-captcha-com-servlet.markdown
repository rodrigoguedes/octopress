---
layout: post
title: "Utilizando Captcha com Servlet"
date: 2014-04-14 10:16:05 -0300
comments: true
categories: [servlet, captcha, segurança] 
---

Um assunto muito interessante que eu trato neste post, é como utilizar captcha para manter seguro seu site contra robôs da web e ataques.

**O que é um Robô da Web**

ROBOT (ou robô) é um programa de computador que percorre automaticamente as páginas da Internet em busca de documentos, a fim de indexá-los, validá-los ou monitorar alterações de conteúdo.
[Wikipedia](http://pt.wikipedia.org/wiki/Robots.txt)

**O que é DDos-Spam**

Um ataque de negação de serviço (também conhecido como DoS, um acrônimo em inglês para Denial of Service), é uma tentativa em tornar os recursos de um sistema indisponíveis para seus utilizadores. Alvos típicos são servidores web, e o ataque tenta tornar as páginas hospedadas indisponíveis na WWW. Não se trata de uma invasão do sistema, mas sim da sua invalidação por sobrecarga.
[Wikipedia](http://pt.wikipedia.org/wiki/Ddos)

**O que é Captcha**

CAPTCHA é um acrônimo para “Completely Automated Public Turing test to tell Computers and Humans Apart” (teste de Turing público completamente automatizado para diferenciar entre computadores e humanos) desenvolvido pela universidade do Carnegie-Mellon.

CAPTCHAs são utilizados para impedir que softwares automatizados execute ações que degradam a qualidade do serviço de um sistema dado, devido à despesa do abuso ou do recurso. Embora CAPTCHAs sejam utilizados mais frequentemente como uma resposta a proteção de interesses comerciais, a noção que existem para parar somente spammers é um erro, ou uma simples redução.
[Wikipedia](http://pt.wikipedia.org/wiki/CAPTCHA)

**API Disponíveis**
Atualmente as apis mais utilizadas, freqüentemente, em sistemas em java são:

Simple Captcha [http://simplecaptcha.sourceforge.net](http://simplecaptcha.sourceforge.net)
Última Atualização 25/09/2005

JCaptcha [http://jcaptcha.sourceforge.net](http://jcaptcha.sourceforge.net)
Última Atualização 03/05/2007

SkewPassImage [http://skewpassim.sourceforge.net](http://skewpassim.sourceforge.net)
Última Atualização 27/03/2006

Recaptcha [http://www.recaptcha.net](http://www.recaptcha.net)

**Implementação**
Uma situação real em que pode ser utilizado a captcha, é em um cadastro de usuário em um fórum, por exemplo, imagine se não existir um método que empeça de um cracker ou um aplicativo robô, derrubar seu site de tanto emitir submissões de cadastros de usuário “Ataque por DDos-spam”. Isso poderia gerar um excessivo uso de recursos desnecessário do servidor.
Para nossa implementação eu escolho o Jcaptcha, pois na minha opinião é a API mais completa para se trabalhar com Java.

{% codeblock CaptchaServiceSingleton.java lang:java %}
package br.com.guedesdesouza.captcha;
 
import com.octo.captcha.service.image.ImageCaptchaService;
import com.octo.captcha.service.image.DefaultManageableImageCaptchaService;
 
public class CaptchaServiceSingleton {
 
    private static ImageCaptchaService instance = new DefaultManageableImageCaptchaService();
 
    public static ImageCaptchaService getInstance(){
        return instance;
    }
}
{% endcodeblock %}


{% codeblock GeraCaptchaServlet.java lang:java %}
package br.com.guedesdesouza.captcha;
 
import java.awt.image.BufferedImage;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
 
import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import com.octo.captcha.service.CaptchaServiceException;
import com.sun.image.codec.jpeg.JPEGCodec;
import com.sun.image.codec.jpeg.JPEGImageEncoder;
 
public class GeraCaptchaServlet extends HttpServlet {

    @Override
    public void init(ServletConfig config) throws ServletException {
	    super.init(config);
	}
 
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)throws ServletException, IOException {
		byte[] captchaChallengeAsJpeg = null;
		// Objeto que receber a imagem jpeg gerada pelo JCaptcha
		// retorna para solicitação do cliente
		ByteArrayOutputStream jpegOutputStream = new ByteArrayOutputStream();
		try {
			// Pega o id da sessao para utiliza como identificador
			// para geração do captcha
			String captchaId = req.getSession().getId();
			// chama o método de geração de captcha
			BufferedImage challenge = CaptchaServiceSingleton.getInstance().getImageChallengeForID(captchaId,req.getLocale());
 
			// codifica a imagem para tipo jpeg
			JPEGImageEncoder jpegEncoder = JPEGCodec.createJPEGEncoder(jpegOutputStream);
			jpegEncoder.encode(challenge);
		} catch (IllegalArgumentException e) {
			resp.sendError(HttpServletResponse.SC_NOT_FOUND);
			return;
		} catch (CaptchaServiceException e) {
			resp.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
			return;
		}
 
		captchaChallengeAsJpeg = jpegOutputStream.toByteArray();
 
		// enviar a resposta com a imagem
		resp.setHeader("Cache-Control", "no-store");
		resp.setHeader("Pragma", "no-cache");
		resp.setDateHeader("Expires", 0);
		resp.setContentType("image/jpeg");
		ServletOutputStream responseOutputStream = resp.getOutputStream();
		responseOutputStream.write(captchaChallengeAsJpeg);
		responseOutputStream.flush();
		responseOutputStream.close();
	}
}
{% endcodeblock %}

{% codeblock ValidaCaptchaRegistroServlet.java lang:java %}
package br.com.guedesdesouza.captcha;
 
import java.io.IOException;
 
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
public class ValidaCaptchaRegistroServlet extends HttpServlet{
 
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp)
		throws ServletException, IOException {
	 
		String resposta = req.getParameter("captcha.id");
		String idCaptcha = req.getSession().getId();
		 
		RequestDispatcher rd = null;
		 
		if (isValido(resposta, idCaptcha)){//se verdadeiro conclui cadastro
			rd = getServletContext().getRequestDispatcher("/concluido.jsp");
			rd.forward(req, resp);
		}else{// caso seja inválido retorna a página de registro
			req.setAttribute("msg","Texto da imagem digitado errado");
			rd = getServletContext().getRequestDispatcher("/registrar.jsp");
			rd.forward(req, resp);
		}
	}
		 
	private boolean isValido(String resposta, String idCaptcha){
		boolean resultado = false;
		try{
			resultado = CaptchaServiceSingleton.getInstance().validateResponseForID(idCaptcha,resposta);
		}catch (Exception e) {
			// TODO: handle exception
		}
		return resultado;
	}
}
{% endcodeblock %}

{% codeblock registro.jsp lang:jsp %}
<?xml version="1.0" encoding="ISO-8859-1" ?>
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
pageEncoding="ISO-8859-1"%>
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="f" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1" />
<style type="text/css">@import url( http://styles/style.css);</style>
<title>Registrar</title>
</head>
<body>
 
<form name="formregister" action="validacaptcha" method="post">
<table class="forumline" cellspacing="1" cellpadding="3" width="50%" border="0" align="center">
<tr>
<th class="thhead" valign="middle" colspan="2" height="25">Registrar Informação</th>
</tr>
 
<tr>
<td class="row2" colspan="2" align="center"><span class="gensmall"><font color="red">Você deve preencher os campos com "*"</font></span></td>
 
</tr>
 
<tr>
<td class="row1" width="38%" align="right">Usuário: *</td>
<td class="row2"><input class="post" type="text" style="WIDTH: 200px" maxlength="25" size="25" name="username" value=""/></td>
</tr>
 
<tr>
<td class="row1" align="right">Endereço de e-mail: *</td>
<td class="row2"><input class="post" type="text" style="WIDTH: 200px" maxlength="255" size="25" name="email" value=""/></td>
 
</tr>
 
<tr>
<td class="row1" align="right">Senha: *</td>
<td class="row2"><input name="password" type="password" class="post" id="password" style="WIDTH: 200px" size="25" maxlength="100" /> </td>
</tr>
 
<tr>
<td class="row1" align="right">Confirme a senha: *</td>
 
<td class="row2"><input class="post" style="WIDTH: 200px" type="password" maxlength="100" size="25" name="password_confirm" /> </td>
</tr>
 
<tr>
<td class="row1" align="right"></td>
<td class="row2"><img src="geracaptcha" /></td>
</tr>
 
<tr>
<td class="row1" align="right">Digite o texto da image: *</td>
 
<td class="row2"><input class="post" type="text" style="WIDTH: 200px" maxlength="7" size="25" name="captcha.id" value=""/></td>
</tr>
 
<tr>
<td class="row2" colspan="2" align="center"><font color="#ff0000"><b>
<c:out value="${msg}"></c:out>
</b></font></td>
</tr>
 
<tr align="center">
<td class="catbottom" colspan="2" height="28">
<input class="mainoption" type="submit" value="Enviar" name="submit" />
<input class="liteoption" type="reset" value="Limpar" name="reset" />
</td>
</tr>
</table>
</form>
</body>
</html>
{% endcodeblock %}

{% codeblock concluido.jsp lang:jsp %}
<?xml version="1.0" encoding="ISO-8859-1" ?>
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
pageEncoding="ISO-8859-1"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1" />
<title>Registro Concluído</title>
</head>
<body>
Registro Concluído...
</body>
</html>
{% endcodeblock %}

{% codeblock web.xml lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app id="WebApp_ID" version="2.4" xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
<display-name>captcha</display-name>
 
<servlet>
<servlet-name>geracaptchaServlet</servlet-name>
<servlet-class>br.com.guedesdesouza.captcha.GeraCaptchaServlet</servlet-class>
<load-on-startup>0</load-on-startup>
</servlet>
 
<servlet>
<servlet-name>validacaptchaServlet</servlet-name>
<servlet-class>br.com.guedesdesouza.captcha.ValidaCaptchaRegistroServlet</servlet-class>
<load-on-startup>0</load-on-startup>
</servlet>
 
<servlet-mapping>
<servlet-name>geracaptchaServlet</servlet-name>
<url-pattern>/geracaptcha</url-pattern>
</servlet-mapping>
 
<servlet-mapping>
<servlet-name>validacaptchaServlet</servlet-name>
<url-pattern>/validacaptcha</url-pattern>
</servlet-mapping>
 
</web-app>
{% endcodeblock %}

**E o resultado é esse:**

![](http://farm9.staticflickr.com/8430/7559645334_9a7ee94868_z_d.jpg)

Apesar de um captcha ser útil para minimizar os problemas de ataque ou vasculhamento de informações por robôs dos grandes motores de buscas da internet e garantir que somente seres humanos possa responder as peguntas geradas, ele também tem uma problema: Como o captcha se baseia na geração de imagens para definição da pergunta, isso se torna um empecilho para usuário com deficiências visuais, pois, para que possam navegar na internet, utilizam-se de software especializados em ler o conteúdo do html e como determinado software não consegue ler a imagem gerada, o deficiente acaba não tendo acesso sobre o conteúdo.Num próximo post irei um pouco mais a fundo do Jcaptcha para podermos utilizar alguns recursos personalizados.
Código Fonte em: [https://github.com/gulira/captcha](https://github.com/gulira/captcha)