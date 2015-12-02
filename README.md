# callslog-contacts
通话记录及联系人CURD


1. get callslog<br>
   通话记录URLCallLog.Calls.CONTENT_URI;<br>
2. get contacts<br>
   联系人查询URL：ContactsContract.CommonDataKinds.Phone.CONTENT_URI<br>
         添加/修改/删除URL:android.provider.ContactsContract.RawContacts.CONTENT_URI<br>
         获取头像URL:ContactsContract.Data.CONTENT_URI<br>
3. get sms<br>
   短信URL：content://sms<br>
       收件箱URL:content://sms/inbox<br>
       发送URL:content://sms/sent<br>
       会话URL:content://sms/conversations<br>
 
1.<1><br>
 //获取通话记录(拨打次数及去重)<br>
 public List<ContactBean> queryCalls(Context context) {
		Uri uri = CallLog.Calls.CONTENT_URI;// 通话记录URI

		List<ContactBean> contactsBeans = new ArrayList<ContactBean>();

		ContentResolver resolver = context.getApplicationContext()
				.getContentResolver();

		String[] projection = new String[] { CallLog.Calls._ID,
				CallLog.Calls.CACHED_NAME, CallLog.Calls.NUMBER,
				CallLog.Calls.DATE, CallLog.Calls.TYPE };
		Cursor cursor = resolver.query(uri, projection, null, null,
				CallLog.Calls.DATE + " DESC");
		if (cursor != null && cursor.getCount() > 0) {
			while (cursor.moveToNext()) {
				long contactId = cursor.getLong(cursor
						.getColumnIndex(CallLog.Calls._ID));
				String displayName = cursor.getString(cursor
						.getColumnIndex(CallLog.Calls.CACHED_NAME));
				String phoneNum = cursor.getString(cursor
						.getColumnIndex(CallLog.Calls.NUMBER));
				long date = cursor.getLong(cursor
						.getColumnIndex(CallLog.Calls.DATE));
				int call_type = cursor.getInt(cursor
						.getColumnIndex(CallLog.Calls.TYPE));

				if (Validationutil.isNullorEmpty(displayName)) {
					Cursor cursor_phone = resolver.query(
							ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
							null, ContactsContract.CommonDataKinds.Phone.NUMBER
									+ "=?", new String[] { phoneNum + "" },
							null);
					if (cursor_phone.moveToFirst()) {
						displayName = cursor_phone.getString(cursor_phone
								.getColumnIndex(Phone.DISPLAY_NAME));
					}
					if (null != cursor_phone) {
						cursor_phone.close();
					}
				}
				if (Validationutil.isNullorEmpty(displayName)) {
					Cursor cursor_name = resolver.query(
							ContactsContract.Data.CONTENT_URI, null,
							ContactsContract.CommonDataKinds.Phone.NUMBER
									+ "=?", new String[] { phoneNum + "" },
							null);
					if (cursor_name.moveToFirst()) {
						displayName = cursor_name
								.getString(cursor_name
									.getColumnIndex(ContactsContract.Data.DISPLAY_NAME));
					}
					if (null != cursor_name) {
						cursor_name.close();
					}
				}

                                //实体类并加入集合中
				ContactBean contact = new ContactBean();
				contact.setContactId(contactId);
				contact.setDisplayName(displayName);
				contact.setPhoneNum(phoneNum);
				contact.setDate(date);
				contact.setCall_type(call_type);
				contactsBeans.add(contact);
			}
		}

		if (null != cursor) {
			<br>cursor.close();
		}
                //检索同一手机号拨打次数<br>
		List<ContactBean> contactsBeans1 = new ArrayList<ContactBean>();
		for (int i = 0; i < contactsBeans.size(); i++) {
			ContactBean cb1 = contactsBeans.get(i);<
			int num = 0;<br>
			for (int j = 0; j < contactsBeans.size(); j++) {
				ContactBean cb2 = contactsBeans.get(j);
				if (cb1.getPhoneNum().equals(cb2.getPhoneNum())) {
					num = num + 1;
				}
			}
			cb1.setCallNum(num);
			contactsBeans1.add(cb1);
		}

                //去重
		Map<String, ContactBean> dataMap = new LinkedHashMap<String, ContactBean>();
		for (ContactBean cb : contactsBeans1) {
			if (!dataMap.containsKey(cb.getPhoneNum())) {
				dataMap.put(cb.getPhoneNum(), cb);
			}
		}
		contactsBeans1 = new ArrayList<ContactBean>(dataMap.values());
		return contactsBeans1;<br>
	}
  <2><br>
  //删除指定通话记录 可通过手机号码/_ID删除<br>
  public boolean deleteCall(Context context, String where,
			String[] selectionArgs) {<br>
		        Uri uri = CallLog.Calls.CONTENT_URI;// 通话记录URI<br>
		        ContentResolver resolver = context.getApplicationContext()
				.getContentResolver();
		        int result = resolver.delete(uri, where, selectionArgs);
		        return result == 0 ? false : true;
	}
<br>2.<1><br>
  //联系人查询<br>
  public List<ContactBean> queryContacts(Context context, String selection,
			String[] selectionArgs) {<br>
		Uri uri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI;// 手机联系人URI<br>

		List<ContactBean> contactsBeans = new ArrayList<ContactBean>();

		ContentResolver resolver = context.getApplicationContext()
				.getContentResolver();
		String[] projection = new String[] {
				ContactsContract.CommonDataKinds.Phone.NUMBER,
				ContactsContract.CommonDataKinds.Phone._ID,
				ContactsContract.CommonDataKinds.Phone.CONTACT_ID,
				ContactsContract.CommonDataKinds.Phone.RAW_CONTACT_ID,
				ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME,
				ContactsContract.CommonDataKinds.Phone.PHOTO_ID };
		Cursor cursor = resolver.query(uri, projection, null, null, null);
		if (cursor != null && cursor.getCount() > 0) {
			while (cursor.moveToNext()) {
				String phoneNumber = cursor
						.getString(cursor
								.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
				if (!Validationutil.isNullorEmpty(phoneNumber)) {<br>
					long id = cursor
							.getLong(cursor
									.getColumnIndex(ContactsContract.CommonDataKinds.Phone._ID));
					long contactId = cursor
							.getLong(cursor
									.getColumnIndex(ContactsContract.CommonDataKinds.Phone.CONTACT_ID));
					long raw_contact_id = cursor
							.getLong(cursor
									.getColumnIndex(ContactsContract.CommonDataKinds.Phone.RAW_CONTACT_ID));
					String name = cursor
							.getString(cursor
									.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));<br>
					long photo_id = cursor
							.getLong(cursor
									.getColumnIndex(ContactsContract.CommonDataKinds.Phone.PHOTO_ID));

					ContactBean contact = new ContactBean();
					contact.setId(id);
					contact.setContactId(contactId);
					contact.setRawContactId(raw_contact_id);
					contact.setPhoneNum(phoneNumber);
					contact.setDisplayName(name);
					contact.setPhotoId(photo_id);
					
					contactsBeans.add(contact);
				}
			}
		}
		return contactsBeans;
	}
  <2><br>
  //添加联系人(与修改联系人类似)<br>
  public String insertContact(Context context, Bitmap bm, String name,
			String number) {<br>
		// 往 raw_contacts 中添加数据，并获取添加的id号<br>
		ContentResolver resolver = context.getApplicationContext()
				.getContentResolver();
		ContentValues values = new ContentValues();
		Uri uri = resolver.insert(RawContacts.CONTENT_URI, values);
		long rawContactId = ContentUris.parseId(uri);
		<br>// 往 data 中添加数据（要根据前面获取的id号）<br>

		<br>// 添加姓名<br>
		values.put(ContactsContract.Data.RAW_CONTACT_ID, rawContactId);
		values.put(ContactsContract.Data.MIMETYPE,
				StructuredName.CONTENT_ITEM_TYPE);
		values.put(StructuredName.DISPLAY_NAME, name);
		resolver.insert(ContactsContract.Data.CONTENT_URI, values);

		<br>// 添加电话<br>
		values.clear();
		values.put(ContactsContract.Data.RAW_CONTACT_ID, rawContactId);
		values.put(ContactsContract.Data.MIMETYPE, Phone.CONTENT_ITEM_TYPE);
		values.put(Phone.NUMBER, number);
		resolver.insert(ContactsContract.Data.CONTENT_URI, values);

		<br>// 添加头像<br>
		try {
			ByteArrayOutputStream baos = new ByteArrayOutputStream();
			bm.compress(Bitmap.CompressFormat.PNG, 100, baos);
			byte[] avatar = baos.toByteArray();

			values.put(ContactsContract.Data.RAW_CONTACT_ID, rawContactId);
			values.put(ContactsContract.Data.MIMETYPE, Photo.CONTENT_ITEM_TYPE);
			values.put(ContactsContract.Contacts.Photo.PHOTO, avatar);
			resolver.insert(ContactsContract.Data.CONTENT_URI, values);
		} catch (Exception e) {
		}
		return String.valueOf(rawContactId);
	}
   <3><br>
   //获取联系人头像<br>
   public Bitmap getContactPortrait(Context context, long raw_contact_id) {<br>
		Bitmap contactPhoto = null;
		ContentResolver resolver = context.getApplicationContext()
				.getContentResolver();

		Cursor cursor = resolver.query(ContactsContract.Data.CONTENT_URI, null,
				ContactsContract.Data.RAW_CONTACT_ID + "=" + raw_contact_id,
				null, null);
		if (null != cursor && cursor.getCount() > 0) {
			while (cursor.moveToNext()) {
				String mimetype = cursor.getString(cursor
						.getColumnIndex(ContactsContract.Data.MIMETYPE));
				byte[] data15 = cursor.getBlob(cursor
						.getColumnIndex(ContactsContract.Contacts.Photo.PHOTO));

				if (null != data15 && data15.length > 0) {
					if (mimetype.equals("vnd.android.cursor.item/photo")) {
						BitmapFactory.Options opt = new BitmapFactory.Options();
						opt.inSampleSize = 1;
						opt.inPurgeable = true;
						opt.inInputShareable = true;
						contactPhoto = BitmapFactory.decodeByteArray(data15, 0,
								data15.length, opt);
					}

				}
			}
		}
		return contactPhoto;
	}

