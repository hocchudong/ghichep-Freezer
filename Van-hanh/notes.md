#Một số ghi chú khi tìm hiểu code

## 1. Điều chỉnh kích thước từng segment upload lên Swift (Byte), thực hiện trên node client
```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/osclients.py

def download_image(self, image):
        """
        Creates a stream for image data
        :param image: Image object for downloading
        :return: stream of image data
        """
        print "Dowload image"
        loader = loading.get_plugin_loader('password')
        auth = loader.load_from_options(
            auth_url="http://172.16.69.130:35357/v2.0",
            username="admin",
            password="Welcome123",
            project_id="8a4ee9793cb34b68b997338cf65dfd55")
        global session
        session = session.Session(auth=auth)
        glance = glance_client('2', session=session)
        print glance.images.get(image.id)
        d = glance.images.data(image.id)
        LOG.debug("Download image enter")
        logging.error('###############################################################')
        return utils.ReSizeStream(d, image.size, 1000000) #Sua thanh kich thuoc mong muon

 ```

## 2. Điều chỉnh kích thước chunk download từ Swift (Byte), thực hiện trên node client
```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/restore.py

def _create_image(self, path, restore_from_timestamp):
        """
        :param path:
        :param restore_from_timestamp:
        :type restore_from_timestamp: int
        :return:
        """
        swift = self.client_manager.get_swift()
        glance = self.client_manager.get_glance()
        backup = self._get_backups(path, restore_from_timestamp)
        path = "{0}_segments/{1}/{2}".format(self.container, path, backup)

        stream = swift.get_object(self.container, "%s/%s" % (path, backup),
                                  resp_chunk_size=10000000) #Sua thanh kich thuoc mong muon
        length = int(stream[0]["x-object-meta-length"])
        LOG.info("Creation glance image")
        image = glance.images.create(
            container_format="bare", disk_format="raw")
        glance.images.upload(image.id,
                             utils.ReSizeStream(stream[1], length, 1))
        return stream[0], image

```
