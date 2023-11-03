本节描述了数据的分类、属性以及存储方式。

# 数据属性

CESS 是一个对象存储服务，所有的数据都包括两种属性：元数据和数据内容。

- **元数据**: 是对数据内容的描述，包括数据长度、数据唯一标识、数据拥有者、数据位置索引等，元数据结构定义如下：<br/>

    ```rust
    pub struct FileInfo<T: Config> {
        pub(super) completion: BlockNumberOf<T>,
        pub(super) stat: FileState,
        pub(super) segment_list: BoundedVec<SegmentInfo<T>, T::SegmentCount>,
        pub(super) owner: BoundedVec<UserBrief<T>, T::OwnerLimit>,
    }

    pub struct SegmentInfo<T: Config> {
        pub(super) hash: Hash,
        pub(super) fragment_list: BoundedVec<FragmentInfo<T>, T::FragmentCount>,
    }

    pub struct FragmentInfo<T: Config> {
        pub(super) hash: Hash,
        pub(super) avail: bool,
        pub(super) miner: AccountOf<T>,
    }

    pub struct UserBrief<T: Config> {
        pub user: AccountOf<T>,
        pub file_name: BoundedVec<u8, T::NameStrLimit>,
        pub bucket_name:  BoundedVec<u8, T::NameStrLimit>,
    }

    pub enum FileState {
        Active,
        Calculate,
        Missing,
        Recovery,
    }
    ```

- **数据内容**: 即数据本身。

# 存储方式

所有数据的元数据会存储到区块链网络中，在存储节点的缓存中也会有备份。数据内容最终都会存储到存储节点中，并且都是以文件的形式存储在磁盘中，文件名字以内容的 hash 值命名，hash 值用[SHA-256](https://wikipedia.org/wiki/SHA-2) 哈希函数计算得出。

下图是展示了存储节点运行后的数据目录结构，该目录位于用户在配置文件中指定的目录下，共分为三级目录结构，一级目录以存储节点的签名账户命名，二级目录固定为 `bucket`，三级目录共包括`db`、`file`、`itag`、`log`、`proof`、`space`、`stag` 及 `tmp` 目录，其中 `space` 目录存储的是闲置数据，`file` 目录存储的是服役数据。

![目录结构](../../assets/ref/storage/dir-structure.png)
