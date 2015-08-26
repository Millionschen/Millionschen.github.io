---
layout: post
title: 自定义一个cursorAdapter
category: 安卓
tags: android
---

## 情景
   QuickContactBadge是一个android组件,可以帮助我们快速跳转到对应的联系人页,还可以显示联系人头像的
thumbnail. 但是在使用QuickContactBadge时,需要将其与联系人的相关信息bind起来,这个时候cursorAdapter就无法
实现了,需要继承cursorAdapter并重写其相关方法.

## 实现
    继承cursorAdapter后必须重写以下方法:
    `CursorAdapter.newView()`
    `CursorAdapter.bindView()`

### newView的作用:
    inflates一个新的view对象来承载listView等的每一项的layout.重写该方法后,就不用为每一项单独的去inflate
一个layout了.
```
@Override
public View newView(
            Context context,
            Cursor cursor,
            ViewGroup viewGroup) {
            /* Inflates the item layout. Stores resource IDs in a
             * in a ViewHolder class to prevent having to look
             * them up each time bindView() is called.
             */
            final View itemView =
                    mInflater.inflate(
                            R.layout.contact_list_layout,
                            viewGroup,
                            false
                    );
            final ViewHolder holder = new ViewHolder();
            holder.displayname =
                    (TextView) view.findViewById(R.id.displayname);
            holder.quickcontact =
                    (QuickContactBadge)
                            view.findViewById(R.id.quickcontact);
            view.setTag(holder);
            return view;
        }

```

    其中viewHold是使用adapter时的一个常用helper类,作用是用于装载layout里每个子view的引用.

### CursorAdapter.bindView()的作用
    这个方法的作用是将cursor中的data放置到每一项的layout包含的view中去
```
@Override
        public void bindView(
                View view,
                Context context,
                Cursor cursor) {
            final ViewHolder holder = (ViewHolder) view.getTag();
            final String photoData =
                    cursor.getString(mPhotoDataIndex);
            final String displayName =
                    cursor.getString(mDisplayNameIndex);
            ...
            // Sets the display name in the layout
            holder.displayname = cursor.getString(mDisplayNameIndex);
            ...
            /*
             * Generates a contact URI for the QuickContactBadge.
             */
            final Uri contactUri = Contacts.getLookupUri(
                    cursor.getLong(mIdIndex),
                    cursor.getString(mLookupKeyIndex));
            holder.quickcontact.assignContactUri(contactUri);
            String photoData = cursor.getString(mPhotoDataIndex);
            /*
             * Decodes the thumbnail file to a Bitmap.
             * The method loadContactPhotoThumbnail() is defined
             * in the section "Set the Contact URI and Thumbnail"
             */
            Bitmap thumbnailBitmap =
                    loadContactPhotoThumbnail(photoData);
            /*
             * Sets the image in the QuickContactBadge
             * QuickContactBadge inherits from ImageView
             */
            holder.quickcontact.setImageBitmap(thumbnailBitmap);
    }
```


### 查询数据的准备
    为了查询数据,还需要做一些准备工作,比如为设置查询哪些项 每一项分别是第几列

```
public class ContactsFragment extends Fragment implements
        LoaderManager.LoaderCallbacks<Cursor> {
...
    // Defines a ListView
    private ListView mListView;
    // Defines a ContactsAdapter
    private ContactsAdapter mAdapter;
    ...
    // Defines a Cursor to contain the retrieved data
    private Cursor mCursor;
    /*
     * Defines a projection based on platform version. This ensures
     * that you retrieve the correct columns.
     */
    private static final String[] PROJECTION =
            {
                Contacts._ID,
                Contacts.LOOKUP_KEY,
                (Build.VERSION.SDK_INT >=
                 Build.VERSION_CODES.HONEYCOMB) ?
                        Contacts.DISPLAY_NAME_PRIMARY :
                        Contacts.DISPLAY_NAME
                (Build.VERSION.SDK_INT >=
                 Build.VERSION_CODES.HONEYCOMB) ?
                        Contacts.PHOTO_THUMBNAIL_ID :
                        /*
                         * Although it's not necessary to include the
                         * column twice, this keeps the number of
                         * columns the same regardless of version
                         */
                        Contacts_ID
                ...
            };
    /*
     * As a shortcut, defines constants for the
     * column indexes in the Cursor. The index is
     * 0-based and always matches the column order
     * in the projection.
     */
    // Column index of the _ID column
    private int mIdIndex = 0;
    // Column index of the LOOKUP_KEY column
    private int mLookupKeyIndex = 1;
    // Column index of the display name column
    private int mDisplayNameIndex = 3;
    /*
     * Column index of the photo data column.
     * It's PHOTO_THUMBNAIL_URI for Honeycomb and later,
     * and _ID for previous versions.
     */
    private int mPhotoDataIndex =
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB ?
            3 :
            0;

```

### 设置listView

我们将listView放在一个fragment中, 这个fragment需要实现LoaderManager.LoaderCallbacks<Cursor>接口
设置listView分为以下几步
1. 在fragment的onCreate中实例化adapter,获取listView的引用
2. 在onActivityCreated中使用listView.setAdapter设置我们的adapter
3. 在onLoadFinished中将传入的用swapCursor 设置传入的cursor
4. 在onLoaderReset中将用swapCursor将cursor设为null, 以释放旧的cursor 加载新的
