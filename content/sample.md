---
title: "Пример страницы"
date: "2015-05-09"
draft: false
---

This is an example page. It's different from a blog post because it will stay in one place and will show up in your site
navigation (in most themes). Most people start with an About page that introduces them to potential site visitors.
It might say something like this:

> Hi there! I'm a bike messenger by day, aspiring actor by night, and this is my blog.
> I live in Los Angeles, have a great dog named Jack, and I like piña coladas. (And gettin' caught in the rain.)

...or something like this:

> The XYZ Doohickey Company was founded in 1971, and has been providing quality doohickeys to the public ever since.
Located in Gotham City, XYZ employs over 2,000 people and does all kinds of awesome things for the Gotham community.

As a new WordPress user, you should go to [your dashboard](http://daybydayz.ru/wp-admin/) to delete this page and
create new pages for your content. Have fun!

```python
@pytest.mark.zfs
def test_create_datapool(node_1, datapool, request):
    """
    Тест создания пула данных (все типы).
    """

    datapool = request.getfixturevalue(datapool)
    check_datapool_on_node(datapool, node_1)
    datapool.delete()
    DataPool.not_exist(datapool.uuid)


def test_default_datapool(random_node):
    """
    Тест получения дефолтного пула данных, создаваемого при добавлении узла в кластер.
    """
    available_types = DataPool.available_types()
    assert available_types
    default_datapool = random_node.get_default_data_pool()
    with pytest.raises(VigilantApiError):
        DataPool.check_verbose_name(default_datapool.name)
    description = 'test_description'
    default_datapool.put(description=description)
    default_datapool.refresh()
    assert default_datapool.data.description == description


def test_clear_datapool(local_datapool):
    """
    Тест полной очистки пула данных.
    """
    assert local_datapool.empty()['empty'] is True
    with pytest.raises(VigilantApiError):
        local_datapool.clear()
    vdisks = []
    for i in range(random.randint(2, 4)):
        vdisk = Vdisk.create(datapool=local_datapool.uuid, size=0.1)
        vdisks.append(vdisk)
    iso = Iso.upload(local_datapool.uuid)
    file = File.upload(local_datapool.uuid)
    assert local_datapool.empty()['empty'] is False
    local_datapool.clear()
    for vdisk in vdisks:
        Vdisk.not_exist(vdisk.uuid)
    Iso.not_exist(iso.uuid)
    File.not_exist(file.uuid)
```

{{< soundcloud-track 205577939 >}}

https://soundcloud.com/mixcult/kirill-matveev-breed
