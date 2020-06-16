# liquibase
Main Liquibase Source
from kazoo.client import KazooClient
from xmlrpc.server import SimpleXMLRPCServer

class ZKConfigDao():

	def __init__(self):
		"""

		初始化方法

		:param connstr: zk的连接串

		:param timeout: zk连接的超时时间，超过改时间未连接上将停止自动重连，单位s

		"""
		self.__connstr = "127.0.0.1:2181"
		# 初始化zkclient
		self.__zkclient = KazooClient(self.__connstr, timeout=10)
		self.timeout = 10
		self.connect()


	def connect(self, ismaster=True):

		"""
		尝试连接zk server
		:param ismaster: 判断是否是主控机在连接, 默认是主控机
		:return:成功返回True，错误返回False

		"""
		try:
			self.__zkclient.start()
			# 连接成功后检查状态
			return True
		except Exception as e:
			print(e)
		return False
	#
	# def __del__(self):
	# 	self.__zkclient.stop()


	#创建路径
	def create(self, path, value):
		if not self.__zkclient.exists(path):
			self.__zkclient.create(path, value=str(value).encode())
			print("create success!")
		else:
			print("path existed!")
			return False


	#查询
	def get(self, path):
		"""
		根据路径获取该路径下的值，如果路径不存在返回None
		:param path:zk中的路径
		:return: 值 or None
		"""

		if self.__zkclient is None or (not self.__zkclient.connected):
			return None
		if not self.__zkclient.exists(path):
			return None
		return self.__zkclient.get(path)

	#修改
	def set(self, path, value):
		"""
		对zk的指定路径的值进行设置
		:param path:zk中的路径
		:param value:值
		:return:True,设置成功；False,设置失败
		"""

		if self.__zkclient is None or (not self.__zkclient.connected):
			return False

		if not self.__zkclient.exists(path):
			return False

		self.__zkclient.set(path, str(value).encode())

		return True

	#删除
	def delete(self, path):
		"""

		删除目录，包括节点下的所有目录

		:param path:zk中的目录

		:return:True 删除成功；False，删除失败

		"""

		try:

			self.__zkclient.delete(path, recursive=True)
			print("delete success!")

		except Exception as e:

			print(e)


	@property
	def zkclient(self):
		"""

		返回zk的连接对象

		:return: zkclient

		"""
		return self.__zkclient




if __name__ == '__main__':

	obj = ZKConfigDao()
	# zkconfigdao.create("/data/data1", "123")
	# rest = zkconfigdao.get("/data/")
	# print(rest)
	#
	# zkconfigdao.set("/data", "hello world!")
	# rest2 = zkconfigdao.get("/data")
	# print(rest2)

	server = SimpleXMLRPCServer(("localhost", 8088))
	# 将实例注册给rpc server
	server.register_instance(obj)

	print("Listening on port 8088")
	server.serve_forever()
	
import xmlrpc.client as xmlrpclib

server = xmlrpclib.ServerProxy("http://localhost:8088")
