package com.isoftstone.sd.framework.controller.printIp;

import java.net.InetAddress;
import java.util.Date;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import com.isoftstone.sd.common.bean.Result;
import com.isoftstone.sd.common.util.LangUtil;
import com.isoftstone.sd.common.util.ResultTool;
import com.isoftstone.sd.framework.bean.PagerDataBean;
import com.isoftstone.sd.framework.bean.printIp.PrinterIPBean;
import com.isoftstone.sd.framework.service.printIp.PrinterIPService;

    /** 
     * 打印配置机信息Controller
     * @author  tanghuibo
     * @see  [相关类/方法]
     * @since  [产品/模块版本]
     */
    @Controller()
    @RequestMapping(value = "/printerIP")
    public class PrinterIPController
    {
        
    	private static final Logger LOGGER = LoggerFactory.getLogger(PrinterIPController.class);
        @Resource
        private PrinterIPService printerIPService;
        
        
        /**
         * 
         * 查询所有的打印机配置信息
         * @param bean
         * @param jquryStylePaging
         * @return
         * @throws Exception
         * @see [类、类#方法、类#成员]
         */
        @RequestMapping(value = "/printerIPList",  method = RequestMethod.POST)
        @ResponseBody
        public Result printerIPList()
            throws Exception
        {
            List<String> printList = printerIPService.getPrintList();
            return ResultTool.successData(printList);
        }
        
        /**
         * 添加 打印配置项
         * @param bean 添加数据
         * @return 操作结果
         * @throws Exception 
         */
        @RequestMapping(value = "/addPrinterIP",  method = RequestMethod.POST)
        @ResponseBody
        public Result addPrinterIP(@RequestBody PrinterIPBean bean)
        {
        	int result = printerIPService.addPrinterIP(bean);
        	if(result == 1) {
        		return ResultTool.fail("IP地址重复");
        	}
        	if(result == 2) {
        		return ResultTool.fail("后台错误");
        	}
        	if(result != 0) {
        		return ResultTool.fail();
        	}
        	return ResultTool.success();
        	
        }
           
        
        /**
         * 修改
         * @param bean 修改数据
         * @return 操作结果
         * @throws Exception 
         */
        @RequestMapping(value = "/updatePrinterIP",  method = RequestMethod.POST)
        @ResponseBody
        public Result updatePrinterIP(@RequestBody PrinterIPBean bean)
        {
        	int result = printerIPService.updatePrinterIP(bean);
        	if(result == 1) {
        		return ResultTool.fail("ip不存在");
        	}
        	if(result == 2) {
        		return ResultTool.fail("后台错误");
        	}
        	if(result != 0) {
        		return ResultTool.fail();
        	}
        	return ResultTool.success();
        }
        
        /**
         * 删除
         * @param gid 主键id
         * @return 操作结果
         * @throws Exception 
         */
        @RequestMapping(value = "/delPrinterIP", method = RequestMethod.POST)
        @ResponseBody
        public Result delPrinterIP(@RequestBody PrinterIPBean bean)
            throws Exception
        {
        	int result = printerIPService.delPrinterIP(bean.getIp());
        	if(result == 1) {
        		return ResultTool.fail("删除失败");
        	}
        	return ResultTool.success();
        }
           
        /**
         * 查询某个打印配置项
         * @param gid 主键id
         * @return 操作结果
         * @throws Exception 
         */
        @RequestMapping(value = "/getPrintConfig", method = RequestMethod.POST)
        @ResponseBody
        public Result getPrintConfig(@RequestBody Map<String,Object> param)
            throws Exception
        {   
            PagerDataBean<PrinterIPBean> result = printerIPService.getPrintConfig(param);
            return ResultTool.successData(result);
        } 
        
        /**
         * 打印预约单
         * @param gid 主键id
         * @return 操作结果
         * @throws Exception 
         */
        @RequestMapping(value = "/printReservations", method = RequestMethod.POST)
        @ResponseBody
        public Result printReservations(@RequestBody Map<String,Object> param, HttpServletRequest httpServletRequest)
            throws Exception
        {   
        	LOGGER.debug("开始打印");
        	String ip = getIpAddress(httpServletRequest);
        	LOGGER.debug("客户端IP:"+ ip);
        	List<Object> list = LangUtil.toBeanList(param.get("appointmentNum"));
        	for (Object object : list) {
				Map<String, Object> map = new HashMap<String, Object>();
				map.put("appointmentNum", object);
				map.put("ip", ip);
				Result result = printerIPService.printReservations(map);
				if(result.getCode() != 1) {
					return result;
				}
			}
            return ResultTool.success();
        } 
        
        
        /**
         * 打印预约单
         * @param gid 主键id
         * @return 操作结果
         * @throws Exception 
         */
        @RequestMapping(value = "/getReservationPic/{reservationNum}", method = RequestMethod.GET)
        public void getReservationPic(@PathVariable("reservationNum") String reservationNum,HttpServletResponse response)
        {   
            printerIPService.getReservationPic(reservationNum, response);
        } 
        
        
        public static String getIpAddress(HttpServletRequest request) {  
        	Enumeration<String> headerNames = request.getHeaderNames();
        	String headerName = "";
        	LOGGER.debug("||====aspect request header====" + new Date());
        	do {
        		if (!"".equals(headerName)) {
        			LOGGER.debug("|| " + headerName + " => "+request.getHeader(headerName));
        		}
        		headerName = headerNames.nextElement();
        	} while (headerName != null);
        	LOGGER.debug("||=====================");
        	
        	
    	    String ip = request.getHeader("x-forwarded-for");
    	    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
    	        ip = request.getHeader("Proxy-Client-IP");
    	    }
    	    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
    	        ip = request.getHeader("WL-Proxy-Client-IP");
    	    }
    	    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
    	        ip = request.getRemoteAddr();
    	        if(ip.equals("127.0.0.1")){
    	            //根据网卡取本机配置的IP
    	            InetAddress inet=null;
    	            try {
    	                inet = InetAddress.getLocalHost();
    	            } catch (Exception e) {
    	                e.printStackTrace();
    	            }
    	            ip= inet.getHostAddress();
    	        }
    	    }
    	    // 多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
    	    if(ip != null && ip.length() > 15){
    	        if(ip.indexOf(",")>0){
    	            ip = ip.substring(0,ip.indexOf(","));
    	        }
    	    }
    	    if (!"0:0:0:0:0:0:0:1".equals(ip)) {
    	        String[] ips = ip.split(":");
    	        ip = ips[ips.length - 1];
    	    }
    	    return getIp(ip);
    	}
        public static String getIp(String ip) {
        	LOGGER.debug("获取到ip==>" + ip);
    		String regex = "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|[1-9])\\." 
                    + "(1\\d{2}|2[0-4]\\ d|25[0-5]|[1-9]\\d|\\d)\\."  
                    + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)\\."  
                    + "(1\\d{2}|2[0-4]\\ d|25[0-5]|[1-9]\\d|\\d)"; 
    	    Pattern p = Pattern.compile(regex); 
            Matcher mm = p.matcher(ip);
            if(mm.find()) { 
                return mm.group(); 
            } else {
            	return ip;
            }
    	}
}
