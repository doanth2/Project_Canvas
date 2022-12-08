pages//

mg list
----
import React, { useRef, useState } from "react";
import Head from 'next/head';
import * as Layout from "../../../components/Layout/index";
import { useDispatch } from "react-redux";
import Grid from '@mui/material/Grid';
import Button from "@mui/material/Button";
import Box from '@mui/material/Box';
import * as store from '../../../stores/Store';

import { format } from 'date-fns' 
import ja from 'date-fns/locale/ja'

import * as commonTips from "../../../constants/CommonTips";
import { StyledHeader, StyledTitleHeader } from "../../../styles/Styled";

import TipsIcon from "../../../components/Common/Tips";
import DataGrid_list from "../../../components/MsgList/Datagrid";
import { InputSelect } from "../../../components/Atoms/Select/Select";
import { useCallClassList } from "../../../features/CallClass/Selector";
import { useInputMsgConditions, useMsgList } from "../../../features/MsgList/Selector";
import { fetchAsyncMsgList } from "../../../features/MsgList/Operation";
import { fetchAsyncCallClassList } from "../../../features/CallClass/Operation";
import msgListSlice from "../../../features/MsgList/Slice";
import { TextField } from "@mui/material";
import { styled } from "@mui/styles";

type OnChangeEvent = React.ChangeEvent<HTMLInputElement>;

export const Main= ():JSX.Element  => {

  // useDispatch で store に紐付いた dispatch が取得できます
  const dispatch: store.AppDispatch = useDispatch();
  const conditions = useInputMsgConditions();
  const callClassList = useCallClassList();
  const msgList = useMsgList();
  const [dispDay, setDispDay] = useState(format(conditions.callDate, 'yyyy/MM/dd'));           // 移動日

  // [スタブ]部署コードが業務の場合はボタン、コンボボックスを非表示
  const busho = "00062";
//  const busho = "00061";

  // useEffectとは、関数の実行タイミングをReactのレンダリング後まで遅らせるhook
  React.useEffect(() => {
    /// 第1引数には実行させたい副作用関数を記述
    /// 第2引数には副作用関数の実行タイミングを制御する依存データを記述
    const promise = async() => {
      await dispatch(fetchAsyncCallClassList(conditions.title));
      await dispatch(fetchAsyncMsgList(conditions));
    };      
    promise();
  },[dispatch])

  // ************************
  // 移動日バリデーション用
  // ************************
  const dispDayValidPattern = "^[0-9/]+$";
  const dispDayRef = useRef<HTMLInputElement>(null);
  const [dispDayError, setDispDayError] = useState(false);

  // ************************
  // バリデーションチェック
  // ************************
  const formValidation = () : boolean => {
    let valid = true;

    // 移動日
    const valDay = dispDayRef?.current;
    if(valDay) {
      const ok = valDay.validity.valid;
      setDispDayError(!ok);
      valid &&= ok;
    }

    return valid;
  };

  // 状態によってボタンテキストを変更
  const completeButtonText = conditions.isShowComplete? "完了分を出さない" : "完了分を出す";
  const autoButtonText = conditions.isShowAuto? "本人送信分を出す" : "自動送信分を出す";
  
  // コール区分コンボボックス
  const handleOnChangeCallClass = (key:string,column:string,value:string | number) => {
    const newConditions = {...conditions,callClass:value.toString()};
    dispatch(msgListSlice.actions.conditionsChanged(newConditions));
  };

  // 本日・最新ボタン
  const handleToday = (newDate:Date) => {
    const newConditions = {...conditions,isToday:true,callDate:newDate,isMove:false};
    dispatch(msgListSlice.actions.conditionsChanged(newConditions)); 

    // 移動日テキストに本日日付
    setDispDay(format(new Date(), 'yyyy/MM/dd'));

    // リクエスト処理
    const promise = async() => {
      await dispatch(fetchAsyncMsgList(newConditions));
    }
    promise();
  };
  
  // 完了分を出すボタン
  const handleIsShowComplete = () => {
    dispatch(msgListSlice.actions.conditionsChanged({...conditions,isShowComplete: !(conditions.isShowComplete)})); 
  };

  // 自動送信分を出すボタン
  const handleIsShowAuto = (value:boolean) => {
    const newConditions = {...conditions,isShowAuto: !(conditions.isShowAuto)};
    dispatch(msgListSlice.actions.conditionsChanged({...conditions,isShowAuto: !(conditions.isShowAuto)})); 

    // リクエスト処理
    const promise = async() => {
      await dispatch(fetchAsyncMsgList(newConditions));
    };
    promise();
  };
  
  // 移動ボタン
  const handleDispDay= (newDate:string) => {
    if (formValidation()){
      var isToday
      var isShow = conditions.isShowComplete
      var isMove = true;
      if(newDate == format(new Date(), 'yyyy/MM/dd'))
      {
        isToday = true;
      } else
      {
        isToday = false;
        isShow = true;
      }

      var valDay = new Date(newDate);

      const newConditions = {...conditions,callDate:valDay, isToday: isToday, isShowComplete: isShow,isMove:isMove};
      dispatch(msgListSlice.actions.conditionsChanged(newConditions));

      // リクエスト処理
      const promise = async() => {
        await dispatch(fetchAsyncMsgList(newConditions));
      }
      promise();
    }
  }

  return(
    <React.Fragment>
      <Head>
        <title>伝言状況</title>
      </Head>
      <br></br>
      <br></br>
      <Grid container alignItems='right' justifyContent='right'>
        <TipsIcon
          title={commonTips.CST_TIPS_TITLE_MSGLIST}
          content={commonTips.CST_TIPS_CONTENT_MSGLIST}
        >
        </TipsIcon>
      </Grid>
      <Grid>
        <StyledHeader>
          <span>
            <strong>
              {format(conditions.callDate, 'M月d日(E)', {locale: ja})}
            </strong>
          </span>
          　
          {/* [スタブ]部署が業務の場合のみコンボボックス表示 */}
          {busho === "00062" &&
            <InputSelect 
              list={callClassList}
              title="区分："
              name="nm1"
              indx="callClassName"
              value="cd1"
              column="callClassName"
              unqkey="cd1"
              defaultValue={conditions.callClass}
              onChange={handleOnChangeCallClass}
              options={{blank:true,all:false}}
            />
          }
        </StyledHeader>
      </Grid>
      <br></br>
      <Box
        sx = {{
          bgcolor: 'info.main',
          color: 'background.paper',
          p: 2 ,
          textAlign: 'center',
          maxWidth : "100%",
        }}
      >
        <StyledTitleHeader>
          <strong>伝言一覧</strong>
        </StyledTitleHeader>
      </Box>
      <Grid>
        <Box
          sx = {{
            height: '90%',
            width: '100%',
          }}
        >
          <DataGrid_list 
            list = {msgList}
          />
        </Box>
      </Grid>
      <br></br>
      <Box
        sx = {{
          height: '100%',
          width: '100%',
        }}
      >
        <Grid container alignItems='center' justifyContent='center'>
          {/* [スタブ]部署が業務の場合のみコンボボックス表示 */}
          {busho === "00062" &&
            <Button
              variant="contained"
              color="primary"
              onClick={() => handleIsShowAuto(conditions.isShowAuto)}
              size='large'
            >
              {autoButtonText}
            </Button>
          }
          　
          <Button
            disabled={!conditions.isToday}
            variant="contained"
            color="primary"
            onClick={() => handleIsShowComplete()}
            size='large'
          >
            {completeButtonText}
          </Button>
          　
          <Button
            variant="contained"
            color="primary"
            onClick={() => handleToday(new Date())}
            size='large'
          >
            本日・最新
          </Button>
          　
            <TextField
              required
              id = "dispDay"
              label = "移動日（yyyy/mm/dd）"
              variant = "filled"
              size = "small"
              inputRef = { dispDayRef }
              inputProps = {{ minLength: 10, maxLength: 10, pattern: dispDayValidPattern }}
              value = {dispDay}
              onChange = {(e: OnChangeEvent) => setDispDay(e.target.value)}
              error = { dispDayError }
              helperText = { dispDayError && dispDayRef?.current?.validationMessage}
            />
          　
          <Button
            variant="contained"
            color="primary"
            onClick={() => handleDispDay(dispDay)}
            size='large'
          >
            移動
          </Button>
          <span>　未完了件数：{msgList.filter(x => x.resultFlg === false).length}件</span>
          {conditions.isMove === true &&
              <span>
                <strong style={{color:"red"}}>
                  　移動日で指定した日付のみ表示しています
                </strong>
              </span>
            }
        </Grid>
      </Box>
    </React.Fragment>
  )
};

export const Index = ():JSX.Element => {
  return (
      <Layout.Index mainComponent={<Main />} title='伝言入力' />
  );
};

export default Index;

----
datagrid
----

import React from 'react'
import { DataGrid, GridColDef, jaJP } from '@mui/x-data-grid';
import { Button, Link as MuiLink } from '@mui/material';
import Box from '@mui/material/Box';
import * as common from '../../constants/Common';
import NextLink from "next/link";
import { StyledDataGrid } from "../../styles/Styled";

const DataGrid_list = (props) => {
  const [pageSize, setPageSize] = React.useState<number>(10);

  // 改行コード置換（\r\nを<BR/>に）
  const replaceContent = (content) => {
    const str = content.split(/(\r\n)/).map((item,index) => {
        return (
            <React.Fragment key={index}>
                { item.match(/\r\n/) ? <br /> : item }
            </React.Fragment>
        );
    });
  
    return <div>{str}</div>
  }
   
  const columns: GridColDef[] = [
    { field: 'rowNo', headerName: 'No.', type:'string', width: 70 , headerAlign: 'center', align: 'center', flex: 0.2},
    { field: 'sendDt', headerName: '送信日時', type:'string', headerAlign: 'center', align: 'center', flex: 0.4 },
    { field: 'operatorNm', headerName: '送信先', type:'string', headerAlign: 'center', align: 'left', flex: 0.5 },
    { field: 'urgent', headerName: '至急', type:'string', headerAlign: 'center', align: 'center', flex: 0.2 },
    { field: 'customerNm', 
      headerName: '顧客名', 
      type:'string', 
      headerAlign: 'center',
      align: 'left',
      flex: 0.5, 
      renderCell: (params) => {
        if(!params.row.datFlg) {
          return <NextLink
                  href={{
                    pathname:"BasicRegist",
                    query:{customerCd :params.row.customerCd}
                  }} passHref as = "BasicRegist"
                 >
                 <MuiLink>{params.row.customerNm}</MuiLink>
                 </NextLink>
        } else {
          return params.row.customerNm
        }
    }},
    { field: 'callTypeTxt', headerName: '分類', type:'string', headerAlign: 'center', align: 'left', flex: 0.6 },
    { field: 'memo',
      headerName: 'メモ',
      type:'string',
      headerAlign: 'center',
      align: 'left',
      flex: 0.8,
      renderCell:(params) => {
        return replaceContent(params.row.memo)
    }},
    { field: 'result', headerName: '状態', type:'string', headerAlign: 'center', align: 'center', flex: 0.2 },
    // 隠し項目
    { field: 'customerCd', headerName: '顧客コード', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'seqNo', headerName: '連番', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'urgentFlg', headerName: '至急フラグ', type:'string', headerAlign: 'center', align: 'left', hide: true },
    { field: 'resultFlg', headerName: '完了フラグ', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'callType', headerName: 'コール分類', type:'string', headerAlign: 'center', align: 'left', hide: true },
    { field: 'callClass', headerName: 'コール区分', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'crtDt', headerName: '作成日', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'wrtDt', headerName: '更新日', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'callDt', headerName: 'コール日付', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'callHour', headerName: 'コール時', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'callMin', headerName: 'コール分', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'datFlg', headerName: '論理削除フラグ', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'defaultFlg', headerName: '初期表示フラグ', type:'string', headerAlign: 'center',align: 'left', hide: true },
  ];

  return (
    <Box
      sx={{
        height: '100%',
        width: '100%',
        // 完了区分（RESULTFLG）が「true：完了」は背景色を変更
        '& .super-app-theme--COMP': {
          bgcolor:common.CST_COLOR_GRAY,
          '&:hover': {
            bgcolor:common.CST_COLOR_HB_GRAY
          }
        },
    }}
    >
      <div>
        <DataGrid getRowId={(row) => row.rowNo}
          sx={StyledDataGrid.grid}
          rows={props.list.map((row) => {
            return {
              ...row,
              rowNo:props.list.indexOf(row)+1
            }
          })}
//          rows={props.list}
          getRowHeight={() => 'auto'}
          columns={columns}
          pageSize={pageSize}
          onPageSizeChange={(newPageSize) => setPageSize(newPageSize)}
          rowsPerPageOptions={[10, 15, 30, 50]}
          pagination
          getRowClassName={(params) => 
            {if(!params.row.resultFlg) {
              return "";
            }
            return `super-app-theme--COMP`}
          }
          localeText={jaJP.components.MuiDataGrid.defaultProps.localeText}
          disableColumnMenu                     // グリッド内のカラムメニュー無効
          disableSelectionOnClick               // 行選択無効
          density='compact'
        />
      </div>
    </Box>
  )
};

export default DataGrid_list;
