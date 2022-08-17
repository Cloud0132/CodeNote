1. 组件开发简要概述
    1. 继承 UnityEditor.Editor
    2. [CustomEditor(Typeof(ClassName))]
    3. 序列化属性 SerializedProperty
    4. GUIContent  显示内容
    5. serializedObject.FindProperty(name), 属性与序列化属性绑定
    6. EditorGUILayout.PropertyField(mItemPrefabDataList, mItemPrefabListContent,false); 序列化属性与显示内容绑定
    7. 示例代码
   ```
    [CustomEditor(typeof(DfLoopListView))]
    public class ListViewEditor : UnityEditor.Editor [CustomEditor(typeof(DfLoopListView))]
    public class ListViewEditor : UnityEditor.Editor
    {
        SerializedProperty mSupportScrollBar;
        SerializedProperty mItemPrefabDataList;

        GUIContent mSupportScrollBarContent = new GUIContent("SupportScrollBar");
        GUIContent mItemPrefabListContent = new GUIContent("ItemPrefabList");
        
        protected virtual void OnEnable()
        {
            mSupportScrollBar = serializedObject.FindProperty("mSupportScrollBar");
            mItemPrefabDataList = serializedObject.FindProperty("mItemPrefabDataList");
        }
        
        void ShowItemPrefabDataList(DfLoopListView listView)
        {
            EditorGUILayout.PropertyField(mItemPrefabDataList, mItemPrefabListContent,false);
            //TODO: isExpanded 应该为true, 但是为false,稍后有时间 查下原因
            // if (mItemPrefabDataList.isExpanded == false)
            // {
            //     return;
            // }
            EditorGUI.indentLevel += 1;
            int removeIndex = -1;
            EditorGUILayout.PropertyField(mItemPrefabDataList.FindPropertyRelative("Array.size"));
            for (int i = 0; i < mItemPrefabDataList.arraySize; i++)
            {
                SerializedProperty itemData = mItemPrefabDataList.GetArrayElementAtIndex(i);
                SerializedProperty mInitCreateCount = itemData.FindPropertyRelative("mInitCreateCount");
                SerializedProperty mItemPrefab = itemData.FindPropertyRelative("mItemPrefab");
                SerializedProperty mItemPrefabPadding = itemData.FindPropertyRelative("mPadding");
                SerializedProperty mItemStartPosOffset = itemData.FindPropertyRelative("mStartPosOffset");
                EditorGUILayout.BeginHorizontal();
                EditorGUILayout.PropertyField(itemData,false);
                if (GUILayout.Button("Remove"))
                {
                    removeIndex = i;
                }
                EditorGUILayout.EndHorizontal();
                if (itemData.isExpanded == false)
                {
                    continue;
                }
                mItemPrefab.objectReferenceValue = EditorGUILayout.ObjectField("ItemPrefab", mItemPrefab.objectReferenceValue, typeof(GameObject), true);
                mItemPrefabPadding.floatValue = EditorGUILayout.FloatField("ItemPadding", mItemPrefabPadding.floatValue);
                if(listView.ArrangeType == ListItemArrangeType.TopToBottom || listView.ArrangeType == ListItemArrangeType.BottomToTop)
                {
                    mItemStartPosOffset.floatValue = EditorGUILayout.FloatField("XPosOffset", mItemStartPosOffset.floatValue);
                }
                else
                {
                    mItemStartPosOffset.floatValue = EditorGUILayout.FloatField("YPosOffset", mItemStartPosOffset.floatValue);
                }
                mInitCreateCount.intValue = EditorGUILayout.IntField("InitCreateCount", mInitCreateCount.intValue);
                EditorGUILayout.Space();
                EditorGUILayout.Space();
            }
            if (removeIndex >= 0)
            {
                mItemPrefabDataList.DeleteArrayElementAtIndex(removeIndex);
            }
            EditorGUI.indentLevel -= 1;
        }

        public override void OnInspectorGUI()
        {
            serializedObject.Update();
            DfLoopListView tListView = serializedObject.targetObject as DfLoopListView;
            if (tListView == null)
            {
                return;
            }
            ShowItemPrefabDataList(tListView);
            EditorGUILayout.Space();
            EditorGUILayout.PropertyField(mSupportScrollBar, mSupportScrollBarContent);
            serializedObject.ApplyModifiedProperties();
        } 
   ```